# Resumen de todo lo que me dio Claude al revisar Backend y Marketplace

Trato de poner lo más importante de todos esos archivos para borrarlos y tener esto como fuente oficial.

> [!Important]
> Todos estos análisis parten del error que causa el feature Copy Form. Está buggeado y cuando se usa Marketplace no es capaz de crear los Progress Forms correspondientes al Intake Form copiado.

# Extracto de Marketplace Form Creation Flow Analysis

> This document traces the complete flow of how Forms with `progress_type = "ongoing"` are created when the `MarketplaceSyncCarePlanWorker` is executed in the backend. The process involves coordination between the backend Rails application and the marketplace Python service.

## MarketplaceSyncCarePlanWorker (Backend)

**Location**: `backend/app/workers/marketplace_sync_care_plan_worker.rb`

**Purpose**: Synchronizes care plan data from backend to marketplace when appointments are created or updated.

**Triggered From**:
- `backend/app/models/episode.rb:453` - After commit callback
- `backend/app/admin/appointments.rb:96` - Manual admin sync
- `backend/app/grimoire/omni/actions/finalize_patient_onboarding.rb:32` - Patient onboarding
- `backend/app/models/episode.rb:1264,1266,1307,1308` - Appointment transfers

**What it does**:
- Calls `marketplace.sync_care_plan_for_scheduling(care_plan.id)`
- Makes PUT request to `/api/v1/internal/sync/patients/{patient_id}/care-plans/{care_plan_id}`
- Sends comprehensive care plan data including appointments, physician info, and insurance data

## Form Creation Process (Marketplace)

### Signal Processing

**Location**: `marketplace/app/marketplace/signals.py:25`

- Emits `CASE_CREATED` signal for new care plans

### Form Listeners

**Location**: `marketplace/app/marketplace/forms/listeners.py:121`

- `LISTENERS_CASE_CREATED = [create_intake_form]`
- Enqueues form creation jobs

### Form Jobs

**Location**: `marketplace/app/marketplace/forms/jobs.py:27-36`
```python
def create_progress_form(aggregate_id: str, appointment_id: str) -> None:
    # Gets care plan and patient data
    forms_controller.create_progress_form(pendulum.now(), patient_result, care_plan_result, appointment_id)
```

### Form Controller (Primary Entry Point)

**Location**: `marketplace/app/marketplace/forms/controller.py:34-87`

**Method**: `create_progress_form()`

### Patient Forms Service (API Communication)

> [!Note]
> Aquí es donde se da el llamado el endpoint que vive en backend.

**Location**: `marketplace/app/marketplace/services/patient_form.py:131-150`

**Method**: `create_progress_form()`

```python
def create_progress_form(intake_form: models.Form) -> entities.FormResponse:
    endpoint = f"{settings.FORMS.API_BASE_URI}forms/{str(intake_form.form_id)}/request_progress_forms"
    json_data = {"forms_attributes": {"type_name": MedicalFormType.from_body_part(intake_form.care_plan_index.affected_body_part)}}
```

## Storage

Marketplace stores Forms in the `patient_form` table with:
- `form_id`: External ID from backend
- `form_type`: "progress" (vs "intake")
- `care_plan_aggregate_id`: Links to care plan
- `triggering_appointment_id`: Which appointment triggered this form
- `trigger_type`: Why this form was created (cadence, pre-op, discharge)
- `status_id`: For checking completion status
- `questions_url`: Link to patient-facing form
- `answers_url`: Link to completed form results

## Key Insight

The `progress_type="ongoing"` field is **not set by the marketplace** - it's set by the **backend** when the marketplace calls the `/forms/{form_id}/request_progress_forms` endpoint. 

**Marketplace's Role**:
- Decide **when** to create progress forms (trigger logic)
- Validate **prerequisites** (completed intake, appointment triggers)
- Make the **API call** to backend
- Store the **result** with trigger metadata for reporting

**Backend's Role**:
- Create the **actual form** with appropriate questions
- Set **progress_type="ongoing"** based on internal logic
- Provide the **patient-facing form URL**
- Handle **form completion and scoring**