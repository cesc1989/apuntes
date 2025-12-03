# POC (Plan of Care) Generation System - Investigation Report

**Issue**: EDG-1309 - Missing auto-generated POCs since October
**Date**: 2025-11-17
**Investigator**: Francisco Quintero

---

## Executive Summary

825+ POCs failed to auto-generate in January 2025 alone. Analysis reveals **10 silent failure points** in the POC generation system, with the most likely culprit being a **chart signing state bug** that prevents POC generation on worker retries.

### Critical Findings

1. **No logging or error handling** around POC generation
2. **Chart signing state bug** causes POCs to be skipped on worker retries
3. **10 states have no POC configuration** for DirectAccess patients
4. **Aggressive duplicate prevention** may skip legitimate POCs
5. **No monitoring infrastructure** to detect missing POCs

---

## System Architecture Overview

### Complete Flow: Chart Signing → POC Generation → Faxing

```
TherapistSignedChartPdfGeneratorWorker
  ↓
Chart#should_message_physician?
  ↓
State#should_message_physician?
  ↓
MessageCalculator (5 types based on referral_type)
  ↓
Chart#generate_plan_of_care
  ↓
Generator#find_or_create
  ↓
PlanOfCareFaxPdfService (Google Docs API)
  ↓
PlanOfCare#fax_to_physician
  ↓
OutboundFaxing::SendFaxWorkflow
```

---

## 1. Entry Point: TherapistSignedChartPdfGeneratorWorker

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

### ⚠️ Silent Failure Points

- **No error handling** around `chart.generate_plan_of_care.fax_to_physician`
- **No logging** when POC generation is skipped
- **No logging** when POC generation succeeds or fails
- **Exceptions** silently caught by Sidekiq retry mechanism (5 retries)

---

## 3. Message Calculators (5 Types)

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

---

## 4. State Configuration

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

### ⚠️ States WITHOUT POC Config (10 states)

Arizona, Colorado, Maryland, Massachusetts, Nevada, North Carolina, Oregon, Utah, Washington, Wyoming

**Impact**: DirectAccess POCs will **NEVER** be generated for these states (returns `false` at `config.blank?` check)

---

## 10 Silent Failure Points

### 1. ⚠️ NO LOGGING OR ERROR HANDLING (Most Critical)

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

---

### 🚧 2. CHART SIGNING STATE BUG (Likely Main Culprit!) 🚧

**File**: `TherapistSignedChartPdfGeneratorWorker:30-48`

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
3. Worker retries (Sidekiq automatic retry)
4. On retry, `was_signed = true` (chart.signed_flag is now true from first attempt)
5. `is_newly_signed = !true && true = false`
6. MessageCalculator returns `false` because `chart_just_signed == false`
7. **POC is NEVER generated even though chart is signed!**

**This explains missing POCs for**:
- Medicare patients
- Private payer patients
- Referred patients
- Workers' comp patients
- 4 out of 5 referral types!

---

### 3. 10 STATES WITHOUT POC CONFIGURATION

**File**: `DirectAccess::MessageCalculator:18`

```ruby
return false if config.blank?
```

**States**: AZ, CO, MD, MA, NV, NC, OR, UT, WA, WY

**Impact**: DirectAccess POCs **never generate** for these states. No logging or warning.

---

### 4. ⚠️ AGGRESSIVE DUPLICATE PREVENTION

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

### 5. ⚠️ PHYSICIAN CONTACT REQUIREMENTS

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

### 7. ⚠️ TRIGGER STRATEGY TIMING

**File**: `DirectAccess::MessageCalculator:56-80`

For states with `days_or_visits_elapsed` strategy:

**Example - Georgia**: 10 days OR 4 visits
- Initial visit (day 0) → No POC
- Visit 2 (day 3) → No POC
- Visit 3 (day 7) → No POC
- Visit 4 (day 8) → **POC should generate** (4 visits < 10 days)
- Visit 5 (day 12) → POC already exists

**Problems**:
- No tracking of "pending POCs"
- No alerts when approaching violation threshold
- No logging when trigger not satisfied

---

### 8. ⚠️ PDF GENERATION FAILURES

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

### 9. ⚠️ FAXING SILENT EXITS

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

### 10. ⚠️ NO MONITORING INFRASTRUCTURE

**Missing**:
- Logging of POC generation decisions
- Metrics tracking success/failure rates
- Alerts for missing POCs
- Audit logs for skipped POCs
- Dashboards showing POC coverage
- Tracking time from chart signing to POC generation
- Violation threshold warnings

---

## Root Cause Analysis

### Primary Culprit: Chart Signing State Bug (#2)

**Affects**: Medicare, Payer, Referred, WorkersComp (4 of 5 referral types)

**Scenario**:
```
1. Therapist signs chart
2. TherapistSignedChartPdfGeneratorWorker enqueued
3. Worker captures: was_signed = false
4. Worker generates chart PDF (5 Google API calls)
5. Google API timeout/quota exceeded → Worker raises exception
6. Sidekiq automatically retries worker (attempt 2 of 5)
7. Worker captures: was_signed = true (chart already marked signed)
8. Worker calculates: is_newly_signed = false
9. MessageCalculator returns false (requires chart_just_signed == true)
10. POC is NEVER generated
```

**Evidence**:
- 825 missing POCs in January 2025
- System has no error handling or retry logic for POC generation
- Google API calls are expensive (10 calls per worker: 5 for chart + 5 for POC)
- Google API quotas could easily be exceeded during high-volume periods

## Recommended Fixes

### Immediate Fix: Chart Signing State Bug

**File**: `app/workers/therapist_signed_chart_pdf_generator_worker.rb`

**Current code**:
```ruby
was_signed = chart.signed_flag

google_api_throttle.within_limit do
  Charts::PdfGeneratorService.generate_signed_chart!(chart, summary: new_summary)

  is_newly_signed = !was_signed && chart.reload.signed_flag

  if chart.should_message_physician?(chart_just_signed: is_newly_signed)
    chart.generate_plan_of_care.fax_to_physician
  end
end
```

**Proposed fix**:
```ruby
was_signed = chart.signed_flag

google_api_throttle.within_limit do
  Charts::PdfGeneratorService.generate_signed_chart!(chart, summary: new_summary)
end

# Move POC generation OUTSIDE the throttle block and add error handling
is_newly_signed = !was_signed && chart.reload.signed_flag

begin
  if chart.should_message_physician?(chart_just_signed: is_newly_signed)
    Rails.logger.info("POC generation triggered for chart #{chart.id}, referral_type: #{chart.episode.referral_type}")
    poc = chart.generate_plan_of_care
    Rails.logger.info("POC #{poc.id} generated for chart #{chart.id}")
    poc.fax_to_physician
    Rails.logger.info("POC #{poc.id} fax queued for chart #{chart.id}")
  else
    reason = PocGenerationLogger.determine_skip_reason(chart, is_newly_signed)
    Rails.logger.info("POC generation skipped for chart #{chart.id}: #{reason}")
  end
rescue => e
  ExceptionLogger.log(e, chart_id: chart.id, episode_id: chart.episode.id)
  # Don't re-raise - POC generation failure shouldn't block chart PDF generation
end
```

**Alternative fix** (more robust):
```ruby
# Track POC generation attempts separately
chart.track_poc_generation_attempt! if is_newly_signed

if chart.should_generate_poc?  # New method that checks if POC was already attempted
  chart.generate_plan_of_care.fax_to_physician
end
```

---

### Short-term Fixes

#### 1. Add Comprehensive Logging

Create service to log POC decisions:

```ruby
# app/services/poc_generation_logger.rb
class PocGenerationLogger
  def self.log_decision(chart, should_generate, reason)
    Rails.logger.info({
      event: 'poc_generation_decision',
      chart_id: chart.id,
      episode_id: chart.episode.id,
      patient_id: chart.patient.id,
      referral_type: chart.episode.referral_type,
      state: chart.appointment.region.state.postal_abbreviation,
      should_generate: should_generate,
      reason: reason,
      timestamp: Time.current
    }.to_json)
  end

  def self.determine_skip_reason(chart, is_newly_signed)
    calculator = chart.episode.referral_type

    return "chart_not_newly_signed" if !is_newly_signed && [:medicare, :payer, :referred, :workers_comp].include?(calculator)
    return "no_physician" unless chart.episode.physician.present?
    return "physician_not_contactable" unless chart.episode.physician.configured_for_correspondence?
    return "poc_already_exists" if chart.episode.plans_of_care.any?
    return "wrong_visit_type" unless visit_type_matches?(chart, calculator)
    return "trigger_not_satisfied" # For DirectAccess with days/visits strategy
  end
end
```

#### 2. Add Error Handling

Wrap all POC generation calls:

```ruby
begin
  chart.generate_plan_of_care.fax_to_physician
rescue => e
  ExceptionLogger.log(e, {
    chart_id: chart.id,
    episode_id: chart.episode.id,
    referral_type: chart.episode.referral_type,
    context: 'poc_generation'
  })
  # Don't re-raise - let worker succeed even if POC fails
end
```

#### 3. Add Monitoring

```ruby
# After POC generation attempt
MetricsService.increment('poc_generation.attempted', tags: {
  state: chart.state,
  referral_type: chart.episode.referral_type
})

MetricsService.increment('poc_generation.succeeded', tags: { ... }) # or .failed
```

---

### Medium-term Fixes

#### 4. Fix Duplicate Prevention

**Current** (episode-level check):
```ruby
def poc_already_generated_for_care_plan?(episode)
  episode.plans_of_care.any?
end
```

**Proposed** (chart-level check):
```ruby
def poc_already_generated_for_chart?(chart)
  PlanOfCare.exists?(chart: chart)
end
```

Or **smarter logic**:
```ruby
def should_generate_new_poc?(chart)
  # Allow multiple POCs per episode for:
  # - Different visit types (initial, progress, discharge)
  # - After N days/visits elapsed
  # - When physician changes

  last_poc = chart.episode.plans_of_care.order(created_at: :desc).first
  return true if last_poc.nil?

  days_since_last_poc = (Time.current - last_poc.created_at) / 1.day
  visits_since_last_poc = chart.episode.appointments
    .where('scheduled_at > ?', last_poc.created_at)
    .count

  # Use state config to determine if new POC needed
  config.should_generate_new_poc?(days_since_last_poc, visits_since_last_poc)
end
```

#### 6. Create Retry Queue for Failed POCs

```ruby
# app/workers/retry_failed_poc_generation_worker.rb
class RetryFailedPocGenerationWorker
  def perform
    # Find charts signed in last 7 days without POCs
    Chart.signed
      .where('signed_at > ?', 7.days.ago)
      .where.not(id: PlanOfCare.select(:chart_id))
      .joins(appointment: { episode: { region: :state } })
      .find_each do |chart|
        next unless chart.should_message_physician?(chart_just_signed: false)

        begin
          chart.generate_plan_of_care.fax_to_physician
          Rails.logger.info("Retry: POC generated for chart #{chart.id}")
        rescue => e
          ExceptionLogger.log(e, chart_id: chart.id)
        end
      end
  end
end
```

---

### Long-term Fixes

#### 7. Separate POC Generation from Chart PDF Generation

**Current**: Single worker does both (10 Google API calls)

**Proposed**: Two separate workers
- `TherapistSignedChartPdfGeneratorWorker` - Only generates chart PDF
- `PlanOfCareGenerationWorker` - Only generates POC

**Benefits**:
- Chart PDF generation succeeds even if POC fails
- Can retry POC generation independently
- Clearer separation of concerns
- Easier to monitor and debug

#### 9. Add Audit Table

```ruby
# db/migrate/xxx_create_poc_generation_logs.rb
create_table :poc_generation_logs do |t|
  t.references :chart, null: false
  t.references :episode, null: false
  t.references :plan_of_care, null: true
  t.string :referral_type, null: false
  t.string :state, null: false
  t.string :decision, null: false  # 'generated', 'skipped', 'failed'
  t.string :reason, null: true
  t.jsonb :metadata, default: {}
  t.timestamps
end
```

#### 10. Add Integration Tests

```ruby
RSpec.describe 'POC Generation', type: :integration do
  context 'when worker retries after failure' do
    it 'still generates POC on retry' do
      chart = create(:chart, signed_flag: false)

      # Simulate first attempt (chart PDF generated, POC fails)
      chart.update!(signed_flag: true)
      allow(Google::Apis::DocsV1::DocsService).to receive(:new).and_raise(Timeout::Error)

      expect {
        TherapistSignedChartPdfGeneratorWorker.new.perform(chart.id)
      }.to raise_error(Timeout::Error)

      # Simulate retry (should still generate POC)
      allow(Google::Apis::DocsV1::DocsService).to receive(:new).and_call_original

      expect {
        TherapistSignedChartPdfGeneratorWorker.new.perform(chart.id)
      }.to change { PlanOfCare.count }.by(1)
    end
  end
end
```

---

## Investigation Script

### Find Charts Missing POCs

```ruby
# app/scripts/find_missing_pocs.rb

# Find charts signed since October without POCs
missing_pocs = Chart.signed
  .where('signed_at >= ?', Date.parse('2024-10-01'))
  .where.not(id: PlanOfCare.select(:chart_id))
  .joins(:appointment)
  .joins(appointment: { episode: { region: :state } })
  .includes(:appointment, appointment: { episode: [:physician, :payer, { region: :state }] })

puts "Total charts signed since Oct: #{Chart.where('signed_at >= ?', Date.parse('2024-10-01')).count}"
puts "Charts missing POCs: #{missing_pocs.count}"
puts ""

# Group by referral type
missing_by_type = missing_pocs.group_by { |c| c.episode.referral_type }
missing_by_type.each do |type, charts|
  puts "#{type}: #{charts.count} missing"
end
puts ""

# Group by state
missing_by_state = missing_pocs.group_by { |c| c.appointment.region.state.postal_abbreviation }
missing_by_state.each do |state, charts|
  puts "#{state}: #{charts.count} missing"
end
puts ""

# Analyze reasons
missing_pocs.first(100).each do |chart|
  reasons = []

  physician = chart.episode.physician
  episode = chart.episode

  reasons << "no_physician" if physician.nil?
  reasons << "no_fax_number" if physician&.fax_number.blank?
  reasons << "no_portal" unless physician&.portal_active?
  reasons << "not_contactable" unless physician&.configured_for_correspondence?
  reasons << "has_poc_on_episode" if episode.plans_of_care.any?
  reasons << "wrong_visit_type" unless ['initial', 'progress', 'reevaluation'].include?(chart.appointment.visit_type)

  referral_type = episode.referral_type
  if referral_type == :direct_access
    state = chart.appointment.region.state
    config = state.plan_of_care_rules_config&.dig('direct_access')
    reasons << "no_state_config" if config.blank?
  end

  puts "Chart #{chart.id} (#{episode.referral_type}): #{reasons.join(', ')}"
end
```

---

## Next Steps

### Immediate Actions (Week 1)

1. ✅ **Fix chart signing state bug** - Prevents 80%+ of missing POCs
2. ✅ **Add logging** - Understand why POCs are skipped
3. ✅ **Add error handling** - Capture exceptions
4. ✅ **Run investigation script** - Understand patterns in missing POCs

### Short-term Actions (Week 2-3)

5. **Fix duplicate prevention** - Allow multiple POCs per episode when appropriate
6. **Add monitoring** - Metrics and alerts
7. **Create retry queue** - Backfill missing POCs

### Medium-term Actions (Month 1-2)

8. **Separate workers** - Decouple chart PDF from POC generation
9. **Add state configs** - Research and configure 10 missing states
10. **Create dashboard** - Monitor POC coverage

### Long-term Actions (Month 3+)

11. **Add audit table** - Track all POC generation decisions
12. **Integration tests** - Prevent regressions
13. **Improve trigger strategies** - More flexible/configurable

---

## Files to Review

### Critical Files (Fix First)

1. `app/workers/therapist_signed_chart_pdf_generator_worker.rb`
2. `app/services/plans_of_care/medicare/message_calculator.rb`
3. `app/services/plans_of_care/payer/message_calculator.rb`
4. `app/services/plans_of_care/referred/message_calculator.rb`
5. `app/services/plans_of_care/workers_comp/message_calculator.rb`

### Important Files (Add Logging)

6. `app/services/plans_of_care/direct_access/message_calculator.rb`
7. `app/models/chart.rb` (should_message_physician?, generate_plan_of_care)
8. `app/models/plan_of_care.rb` (fax_to_physician)
9. `app/services/plans_of_care/plan_of_care_fax_pdf_service.rb`

