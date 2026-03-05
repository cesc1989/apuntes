# Copy Forms Logic Deep Dive

## Overview

The copy forms functionality allows administrators to copy data from a **completed patient form** to an *incomplete form of the same type*, reducing data entry time and potential errors for patients with similar conditions across multiple care plans.

## Complete Flow Analysis

### 1. **UI Selection Logic**(`app/views/admin/patients/manage_forms.html.arb` — `copy_from_form_targets_for_form`)

The `copy_from_form_targets_for_form` helper method determines which forms can be copied from:

```ruby
def copy_from_form_targets_for_form(form, forms)
  forms.select do |f|
    f.completed? &&
    @care_plan_number_by_id[f.care_plan_id] < @care_plan_number_by_id[form.care_plan_id] &&
    f.form_type_id == form.form_type_id &&
    f.progress_type == form.progress_type
  end.sort_by(&:created_at)
end
```

Called as: `copy_from_form_targets_for_form(form, patient.forms)`

**Key Filtering Rules:**
- **Completion requirement**: Only completed forms can be source forms (`f.completed?`)
- **Temporal restriction**: Only forms from prior care plans (lower care plan numbers)
- **Type matching**: Must be exact same form type (`form_type_id`)
- **Progress matching**: Must be same progress type (onboarding vs ongoing)
- **Result ordering**: Sorted by `created_at` ascending

### 2. **UI Form Generation** (`app/views/admin/patients/manage_forms.html.arb` — "Copy from Similar Form" column)

For each incomplete form, generates a dropdown with eligible source forms:

```ruby
# Build dropdown options with descriptive labels
options =
  if similar_forms.present?
    similar_forms.map do |f|
      created_at =
        DateTimeLocalizer.new(f.created_at, patient.region).american_with_hour_and_minute
      [
        "Care Plan ##{@care_plan_number_by_id[f.care_plan_id]} - "\
        "#{f.id} - "\
        "#{f.form_type.acronym.upcase.dasherize} #{progress_type_title(f)} - "\
        "created #{created_at}".html_safe,
        f.id  # This becomes the source_form_id
      ]
    end.unshift(["Select a form to copy from", nil])
  else
    [["No similar forms found", nil]]
  end

# Generate form with authenticity token, hidden target_form_id, dropdown for source_form_id, and submit
form(action: copy_form_admin_patient_path(resource), method: "post") do |web_form|
  web_form.input type: "hidden", name: "authenticity_token", value: form_authenticity_token
  web_form.input type: "hidden", name: "target_form_id", value: form.id
  web_form.select(name: "source_form_id", disabled: options.one?) do
    options.each do |option|
      web_form.option(value: option[1]) { option[0] }
    end
  end
  web_form.input(
    type: "submit",
    value: "Copy Form",
    disabled: options.one?,
    class: options.one? ? "disabled" : ""
  )
end
```

**UX Features:**
- Descriptive labels showing care plan number, form ID, form type, and creation date
- Dropdown and submit button both disabled if no similar forms found
- Shows "No similar forms found" placeholder when no eligible forms exist

### 3. **Controller Action** (`app/admin/customers/patients.rb` — `member_action :copy_form`)

```ruby
member_action :copy_form, method: :post do
  begin
    # Initialize service with form IDs from request
    copier = PatientSelfReport::CopyFormDataFromForm.new(
      params[:source_form_id],
      params[:target_form_id]
    )

    # Check for validation errors
    if copier.errors.present?
      flash[:alert] = copier.errors
    else
      # Execute the copy operation
      copier.copy_form_data_from_form
      flash[:notice] = "Form copied successfully"
    end
  rescue StandardError => e
    # Log and handle unexpected errors
    ExceptionLogger.log(e, "Error copying form: #{e}")
    flash[:alert] = "Please contact #eng-helpdesk - something went wrong."
  end

  # Always redirect back to forms management page
  redirect_to manage_forms_admin_patient_path(resource)
end
```

### 4. **Core Service Class** (`app/models/patient_self_report/copy_form_data_from_form.rb`)

#### **Initialization & Validation** (`#initialize` and `#run_checks`)

```ruby
def initialize(source_form_id, target_form_id)
  @source_form = Form.find_by(id: source_form_id)
  @target_form = Form.find_by(id: target_form_id)
  @errors = []
  run_checks
end

def run_checks
  # Basic existence checks
  @errors << "Source form not found" unless @source_form
  @errors << "Target form not found" unless @target_form

  # Care plan presence checks
  @errors << "Source form must have a care plan" if @source_form.present? && @source_form.care_plan_id.nil?
  @errors << "Target form must have a care plan" if @target_form.present? && @target_form.care_plan_id.nil?

  # Business rule validations
  @errors << "Target form is already completed" if @target_form.present? && @target_form.completed?

  # Data integrity checks (only if both forms exist)
  unless @source_form.nil? || @target_form.nil?
    unless @source_form.patient.id == @target_form.patient.id
      @errors << "Source form patient does not match target form patient"
    end
    unless @source_form.type_name == @target_form.type_name
      @errors << "Source form type does not match target form type"
    end
  end

  # Form type whitelist validation
  unless @source_form.present? && VALID_FORM_TYPES.include?(@source_form.type_name)
    @errors << "Invalid source form type"
  end
  return if @target_form.present? && VALID_FORM_TYPES.include?(@target_form.type_name)

  @errors << "Invalid target form type"
end
```

**Validation Rules:**
- **Existence**: Both forms must exist in database
- **Care plan presence**: Both forms must be associated with a care plan
- **Completion**: Target form must not already be completed
- **Patient matching**: Both forms must belong to same patient (via `patient.id`)
- **Type matching**: Both forms must be same type (double-checking UI logic)
- **Whitelist**: Only approved form types can be copied

**Approved Form Types:**
- ASES, LEFS, NDI, ODI, PSFS, HOOS, KOOS, QUICK_DASH, KOS_ADL, MDQ

#### **Data Copying Logic** (`#copy_form_data_from_form`)

The `copy_form_data_from_form` method performs a comprehensive deep copy:

**1. Basic Form Attributes**

> [!Important]
> Key here are lines copying `completed` and `completed_at` attributes.

```ruby
# Score attributes - calculated values from form responses
target_form.answers_score = source_form.answers_score
target_form.customer_satisfaction_score = source_form.customer_satisfaction_score
target_form.hoos_koos_score = source_form.hoos_koos_score
target_form.pain_scale = source_form.pain_scale
target_form.psfs_score = source_form.psfs_score

# Completion metadata
target_form.submitted_from = source_form.submitted_from
target_form.completed = source_form.completed
target_form.completed_at = source_form.completed_at
target_form.persist_draft_until = source_form.persist_draft_until
```

**2. Aggravating Activities**
```ruby
# Create new AggravatingActivity records linked to target form
target_aggravating_activities = source_form.aggravating_activities.map do |aggravating_activity|
  AggravatingActivity.new(
    name: aggravating_activity.name,
    ability: aggravating_activity.ability,  # 0-10 scale
    form: target_form
  )
end
target_form.aggravating_activities = target_aggravating_activities
```

**3. Form Answers**
```ruby
# Create new Answer records with proper associations
target_form_answers = source_form.answers.includes(:question, :option_choice).map do |source_answer|
  Answer.new(
    content: source_answer.content,        # Text content
    value: source_answer.value,            # Numeric value
    question: source_answer.question,      # Same question reference
    option_choice: source_answer.option_choice,  # Same option reference
    form: target_form                      # Link to target form
  )
end
target_form.answers = target_form_answers
```

**4. Intake Form (Complex Nested Data)**

Only processes if source form has an intake form:

```ruby
if source_form.intake_form.present?
  # Create new intake form with copied attributes (excluding IDs)
  new_intake_form = PatientSelfReport::Forms::BuildIntake.call(
    target_form,
    source_form.intake_form.attributes.except("id")
  )

  # Copy medications (with validation)
  source_form.intake_form.medications.each do |medication|
    next if medication.name.blank? || medication.dosage.blank?
    new_intake_form.medications.build(
      medication.attributes.except("id", "intake_form_id", "created_at", "updated_at")
    )
  end

  # Copy surgeries (with validation)
  source_form.intake_form.surgeries.each do |surgery|
    next if surgery.name.blank? || surgery.date.blank?
    new_intake_form.surgeries.build(
      surgery.attributes.except("id", "intake_form_id", "created_at", "updated_at")
    )
  end

  # Copy diseases (through join table)
  source_form.intake_form.intake_form_diseases.each do |disease|
    new_intake_form.intake_form_diseases.build(
      disease.attributes.except("id", "intake_form_id", "created_at", "updated_at")
    )
  end

  # Copy pain spots (through join table)
  source_form.intake_form.intake_form_pain_spots.each do |pain_spot|
    new_intake_form.intake_form_pain_spots.build(
      pain_spot.attributes.except("id", "intake_form_id", "created_at", "updated_at")
    )
  end

  new_intake_form.save!
  target_form.intake_form = new_intake_form
end

# Save the complete target form with all associations
target_form.save!
```

## Data Structure Deep Dive

### **Core Form Attributes Copied:**
- **Scores**: `answers_score`, `customer_satisfaction_score`, `hoos_koos_score`, `pain_scale`, `psfs_score`
- **Metadata**: `submitted_from`, `completed`, `completed_at`, `persist_draft_until`

### **Associated Data Copied:**

#### **1. AggravatingActivities**
- Patient-specific activities that aggravate their condition
- Each has `name` (activity) and `ability` (0-10 functional scale)
- Creates completely new records linked to target form

#### **2. Answers**
- Individual responses to form questions
- Links to existing `Question` and `OptionChoice` records
- Contains both `content` (text) and `value` (numeric) data
- Creates new records with same question/option references

#### **3. IntakeForm** (if present)
Complex medical history including:

**Basic Info:**
- Primary care physician details
- Symptom onset dates
- Allergy information
- Quality of life assessment

**Medical History Collections:**
- **Medications**: `name` + `dosage` (skips if either blank)
- **Surgeries**: `name` + `date` (skips if either blank)
- **Diseases**: Links to existing disease records via join table
- **Pain Spots**: Links to existing pain spot records via join table

## Business Rules & Safety Features

### **1. Form Type Restrictions**
Only specific, validated form types can be copied:
- Assessment scales: ASES, LEFS, NDI, ODI, PSFS
- Joint-specific: HOOS, KOOS
- Functional: QUICK_DASH, KOS_ADL
- Disability: MDQ

### **2. Temporal Restrictions**
- Can only copy FROM earlier care plans TO later care plans
- Prevents circular copying or copying "backwards" in time
- Maintains logical progression of patient care

### **3. Data Integrity Safeguards**
- Care plan presence check ensures both forms are tied to a care plan
- Patient ID matching ensures no cross-patient contamination (via `patient.id` association)
- Form type matching prevents incompatible data mixing
- Completion status prevents overwriting finished forms
- Extensive error handling with user-friendly messages

### **4. Selective Data Copying**
- Skips blank medication/surgery entries to avoid empty records
- Preserves original question/option associations
- Creates new IDs for all copied records
- Excludes system timestamps from copying

### **5. Transaction Safety**
- All operations wrapped in database transaction via `save!`
- Rollback on any failure prevents partial copying
- Comprehensive error logging for debugging

## Error Handling

### **Validation Errors**
- Form existence, care plan presence, patient matching, type matching
- Displayed to user via flash messages
- User can retry with different selections

### **Runtime Errors**
- Database errors, service failures, unexpected exceptions
- Logged via `ExceptionLogger.log`
- Generic error message directs user to eng support
- Prevents system crashes and data corruption

### **UI Error States**
- Dropdown and submit button disabled when no valid source forms found
- "No similar forms found" placeholder shown instead of empty dropdown
- Graceful degradation for edge cases

## Performance Considerations

### **Database Optimization**
- Uses `includes(:question, :option_choice)` to prevent N+1 queries
- Single transaction for all related record creation
- Efficient filtering in Ruby rather than complex SQL

### **Memory Usage**
- Processes one target form at a time
- Creates new objects incrementally
- Garbage collects original form references

This copy forms system provides a robust, safe, and user-friendly way to duplicate patient form data while maintaining strict data integrity and providing comprehensive error handling.
