# Reglas para la creación de los Progress Forms en Marketplace

En palabras de Claude:

> The Marketplace service contains sophisticated logic for creating Progress Forms based on appointment completion events, with intelligent cadence management, surgical workflow support, and duplicate prevention.

## Key Configuration Settings

**Form Cadence Settings** (`app/marketplace/config/base.py`)

En `class FormsSettings:`

- `PROGRESS_FORM_EVERY_N_VISITS: int = 5` - Progress form sent after the 4th appointment is completed (N-1)
- `PROGRESS_FORM_MINIMUM_N_FINISHED_VISITS: int = 2` - Never send if fewer than 2 visits completed
- `PROGRESS_NOTE_EVERY_N_VISITS: int = 10`

## Main Trigger Events and Listeners

### Primary Trigger - Visit Finished

**File**: `app/marketplace/forms/listeners.py`

- **Event**: `TherapistStartedSession` (Note: Despite the name, this actually triggers on visit completion)
- **Listener**: `LISTENERS_VISIT_FINISHED = [create_progress_form]`
- **Job**: Enqueues `jobs.create_progress_form` with appointment_id and aggregate_id

### Secondary Triggers

**File**: `app/marketplace/forms/listeners.py`

- **Visit Discharged**: `LISTENERS_VISIT_DISCHARGED = [create_final_progress_form]`
- **Intake Completed**: `LISTENERS_INTAKE_FORM_COMPLETED = [create_skipped_progress_forms]`

## Progress Form Creation Logic

### Algorithm

**Location**: `app/marketplace/es/aggregates.py:439`

The `expected_progress_form_trigger_appointments()` method determines when progress forms should be created based on:

#### Trigger Types (`marketplace.constants.FormTriggerType`)

En `app/marketplace/constants.py`.

**Collection Cadence** (`FormTriggerType.COLLECTION_CADENCE`): 
   - Every 5th appointment (configurable)
   - Formula: `(effective_appointment_number + 1) % PROGRESS_DUE_EVERY_N_VISITS == 0`

**Pre-Op Visit** (`FormTriggerType.PRE_OP_VISIT_COMPLETED`):
   - When patient had surgery and has post-op visits
   - Only analyzed retrospectively

**Discharge Visit** (`FormTriggerType.DISCHARGE_VISIT_COMPLETED`):
   - When appointment type is "discharge"

#### Prerequisites for Form Creation

- Must have completed at least 2 appointments (`PROGRESS_FORM_MINIMUM_N_FINISHED_VISITS`)
- Intake Form must be completed 
- No existing Progress Form for the same appointment
- Haven't exceeded expected number of forms for the care plan

## Proceso de Creación de Progress Form

Se da como describo en [[Resumen de la Info que recopiló Claude]]

## Key Differences from Backend Logic

**Marketplace vs Backend Comparison**

| Aspect | Backend | Marketplace |
|--------|---------|-------------|
| **Triggering** | Simple appointment completion events | Complex cadence logic (every 5th visit) |
| **Surgical Support** | None | Handles pre-op/post-op form timing |
| **Trigger Metadata** | Basic form creation | Tracks WHY each form was created |
| **Skip Prevention** | None | Creates missed forms when intake completed |
| **Form Limits** | None | Prevents creating too many forms per care plan |