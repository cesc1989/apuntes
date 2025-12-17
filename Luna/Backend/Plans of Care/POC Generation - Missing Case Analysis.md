# POC (Plan of Care) Generation System - Investigation Report

**Issue**: EDG-1309 - Missing auto-generated POCs since October
**Date**: 2025-11-17

---

## Summary

825+ POCs failed to auto-generate in January 2025 alone. Analysis reveals **10 silent failure points** in the POC generation system, with the most likely culprit being a **chart signing state bug** that prevents POC generation on worker retries.

## Entry Point: TherapistSignedChartPdfGeneratorWorker

**File**: `app/workers/therapist_signed_chart_pdf_generator_worker.rb:14-58`

### Key Logic
```ruby
def perform(chart_id, force_generate = false)
  chart = Chart.includes(...).find(chart_id)

  return unless chart.signed?  # ⚠️ SILENT EXIT #1

  was_signed = chart.signed_flag

  return unless force_generate == true ||
                chart.signed_path.blank? ||
                chart.signed_flag == false ||
                chart.summary_changed?(new_summary)  # ⚠️ SILENT EXIT #2

  google_api_throttle.within_limit do
    Charts::PdfGeneratorService.generate_signed_chart!(chart, summary: new_summary)

    is_newly_signed = !was_signed && chart.reload.signed_flag

    # THIS IS WHERE POC GENERATION HAPPENS
    if chart.should_message_physician?(chart_just_signed: is_newly_signed)
      chart.generate_plan_of_care.fax_to_physician
    end
  end
end
```

## State Configuration

**Location**: `db/seeds/states/*.yml`

### Example: Georgia

```yaml
plan_of_care_rules_config:
  direct_access:
    fax_rules:
      anchor: initial_visit
      trigger:
        strategy: days_or_visits_elapsed
        days: 10
        visits: 4
    violation_rules:
      pending:
        days_elapsed: 15
        visits_elapsed: 6
      violation:
        days_elapsed: 21
        visits_elapsed: 8
    resolution_mode: referral  # "referral" | "plan_of_care" | "functional_progress"
```

###  🟡 States WITHOUT POC Config (10 states) 🟡

- Arizona
- Colorado
- Maryland
- Massachusetts
- Nevada
- North Carolina
- Oregon
- Utah
- Washington
- Wyoming

**Impact**: DirectAccess POCs will **NEVER** be generated for these states (returns `false` at `config.blank?` check).

---

## Message Calculators (5 Types)

### A. DirectAccess::MessageCalculator

**File**: `app/services/plans_of_care/direct_access/message_calculator.rb:13-26`

```ruby
def should_message_physician?(chart, chart_just_signed: false)
  appointment = chart.appointment
  episode = chart.episode
  physician = episode.physician

  return false if config.blank?  # ⚠️ State has no POC config
  return false unless contactable_physician?(physician)  # ⚠️ No fax/portal
  return false if poc_already_generated_for_care_plan?(episode)  # ⚠️ Already has POC
  return true if physician.fax_every_visit?  # Always send if enabled
  return true if appointment.discharged?  # Always send on discharge
  return false unless matching_appointment_type?(appointment)  # ⚠️ Not initial/progress

  satisfies_trigger_strategy?(chart, chart_just_signed: chart_just_signed)
end
```

#### Trigger Strategies

1. **`chart_signature`**: Immediate trigger when chart newly signed
   - Used in: CT, KS, OH
2. **`days_elapsed`**: Triggers after N days from anchor visit
   - Used in: TX (15 days)
3. **`days_or_visits_elapsed`**: Triggers after N days OR N visits
   - Used in: CA (35 days/10 visits), GA (10 days/4 visits), etc.

### B. Medicare::MessageCalculator

**File**: `app/services/plans_of_care/medicare/message_calculator.rb:9-21`

```ruby
def should_message_physician?(chart, chart_just_signed: false)
  return false unless chart.signed? && chart_just_signed == true  # ⚠️ Must be newly signed
  return false unless contactable_physician?(physician)
  return false if physician_already_notified?(chart, physician)
  return true if physician.fax_every_visit?
  return true if appointment.discharged?

  matching_appointment_type?(appointment)  # initial, progress, or reevaluation
end
```

**Key Difference**: Always requires `chart_just_signed == true` (not configurable)

### C. Referred::MessageCalculator

**File**: `app/services/plans_of_care/referred/message_calculator.rb`

Same logic as Medicare (requires newly signed chart).

### D. Payer::MessageCalculator

**File**: `app/services/plans_of_care/payer/message_calculator.rb`

Same logic as Medicare (requires newly signed chart).

### E. WorkersComp::MessageCalculator

**File**: `app/services/plans_of_care/workers_comp/message_calculator.rb`

Same as Medicare but does NOT automatically send on discharge.


## Failure Points

### 1. NO LOGGING OR ERROR HANDLING 🟢

**File**: `TherapistSignedChartPdfGeneratorWorker:48`

```ruby
if chart.should_message_physician?(chart_just_signed: is_newly_signed)
  chart.generate_plan_of_care.fax_to_physician  # NO ERROR HANDLING, NO LOGGING
end
```

**Problems**:
- No logging when `should_message_physician?` returns false
- No logging when POC generation succeeds
- No error handling if generation/faxing fails
- Exceptions caught by Sidekiq retry (5 attempts, then silent failure)

### 2. CHART SIGNING STATE BUG ❌

**File**: `TherapistSignedChartPdfGeneratorWorker`

```ruby
was_signed = chart.signed_flag  # BEFORE PDF generation

# ... generate PDF ...

is_newly_signed = !was_signed && chart.reload.signed_flag

if chart.should_message_physician?(chart_just_signed: is_newly_signed)
  chart.generate_plan_of_care.fax_to_physician
end
```

**File**: Medicare/Payer/Referred/WorkersComp MessageCalculators

```ruby
return false unless chart.signed? && chart_just_signed == true
```

**THE BUG**:
1. Worker runs, `was_signed = false`
2. PDF generation fails (Google API timeout, quota, etc.)
3. ~~Worker retries (Sidekiq automatic retry)~~
4. ~~On retry, `was_signed = true` (chart.signed_flag is now true from first attempt)~~
5. ~~`is_newly_signed = !true && true = false`~~
6. ~~MessageCalculator returns `false` because `chart_just_signed == false`~~
7. ~~**POC is NEVER generated even though chart is signed!**~~

> [!Warning]
> Esto no pasa porque si `Charts::PdfGeneratorService.generate_signed_chart!` falla el campo `signed_flag` ni `signed_path` se actualizan. Así que `is_newly_signed` se mantendría como `true`.


### 4. AGGRESSIVE DUPLICATE PREVENTION

**File**: `DirectAccess::MessageCalculator:20`

```ruby
return false if poc_already_generated_for_care_plan?(episode)
```

**File**: `app/services/plans_of_care/direct_access/message_calculator.rb:82-84`

```ruby
def poc_already_generated_for_care_plan?(episode)
  episode.plans_of_care.any?  # Checks ENTIRE episode, not just this chart
end
```

**Problem**: If episode has ANY POC (even from a different chart), new POCs won't generate.

**Example**:
- Initial visit (Chart 1) → POC generated ✓
- Progress visit (Chart 2) → POC NOT generated (episode already has POC)
- Discharge visit (Chart 3) → POC NOT generated (episode already has POC)

---

### 5. PHYSICIAN CONTACT REQUIREMENTS

**File**: All MessageCalculators

```ruby
return false unless contactable_physician?(physician)
```

**File**: `DirectAccess::MessageCalculator:44-46`

```ruby
def contactable_physician?(physician)
  physician.present? && physician.configured_for_correspondence?
end
```

**File**: `app/models/physician.rb:175-177`

```ruby
def configured_for_correspondence?
  portal_active? || fax_number.present?
end
```

**Fails silently when**:
- `physician.nil?`
- `physician.fax_number.blank? && !physician.portal_active?`
- No logging of WHY physician is not contactable

---

### 6. VISIT TYPE RESTRICTIONS

**File**: All MessageCalculators

```ruby
return false unless matching_appointment_type?(appointment)
```

**Different types per calculator**:
- **DirectAccess/Referred/WorkersComp**: `initial` OR `progress` only
- **Medicare/Payer**: `initial` OR `progress` OR `reevaluation`

**Other visit types silently skipped** (no logging):
- Standard visits
- Reassessment visits
- Follow-up visits
- Etc.

---

### 8. PDF GENERATION FAILURES

**File**: `app/services/plans_of_care/plan_of_care_fax_pdf_service.rb:92-111`

### Google API Workflow (5 API calls)

1. Copy template file
2. Batch update document (field replacements)
3. Export as PDF
4. Attach to patient
5. Delete template copy

**Silent failures**:
- Google API quota exceeded
- Template ID not found
- Field replacement errors
- PDF export timeout
- Network failures
- Transaction rollback (no logging)

---

### 9. FAXING SILENT EXITS

**File**: `app/models/plan_of_care.rb:230-239`

```ruby
def fax_to_physician
  return if physician.portal_active? && mode_of_signature != :fax  # ⚠️ Portal physicians
  return if physician.nil? || physician.fax_number.blank? || document.nil?  # ⚠️ Missing data
  return if plan_of_care_faxes.any? { |f| f.physician_id == physician.id }  # ⚠️ Already faxed

  PlanOfCareFax.queue!(plan_of_care: self, document: document, physician: physician)
end
```

**No logging when fax skipped because**:
- Physician has active portal (should use portal instead)
- Missing physician, fax number, or document
- POC already faxed to this physician

---

### 10. NO MONITORING INFRASTRUCTURE

**Missing**:
- Logging of POC generation decisions
- Metrics tracking success/failure rates
- Alerts for missing POCs
- Audit logs for skipped POCs
- Dashboards showing POC coverage
- Tracking time from chart signing to POC generation
- Violation threshold warnings
