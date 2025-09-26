# Reglas para la creación de los Progress Forms en Marketplace

Etiquetas: #luna_help_desk 

En palabras de Claude:

> The Marketplace service contains sophisticated logic for creating Progress Forms based on appointment completion events, with intelligent cadence management, surgical workflow support, and duplicate prevention.

## Key Configuration Settings

**Form Cadence Settings** (`app/marketplace/config/base.py`)

En `FormsSettings:`

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


# Ejemplos de Cadencia de Creación de Progress Forms

Le pedí a Claude que según una serie de visitas diera ejemplos de cuándo se crearía el Progress Form.

**Example 1: Regular Treatment (No Surgery)**

```
Appointment 1 (Initial)     → No form (initial visit, < 2 completed)
Appointment 2 (Progress)    → No form (< 5 visits)
Appointment 3 (Progress)    → No form (< 5 visits)
Appointment 4 (Progress)    → No form (< 5 visits)
Appointment 5 (Progress)    → ✅ PROGRESS FORM (cadence: 5th appointment)
Appointment 6 (Progress)    → No form (< 10 visits)
Appointment 7 (Progress)    → No form (< 10 visits)
Appointment 8 (Progress)    → No form (< 10 visits)
Appointment 9 (Progress)    → No form (< 10 visits)
Appointment 10 (Progress)   → ✅ PROGRESS FORM (cadence: 10th appointment)
Appointment 11 (Progress)   → No form (< 15 visits)
Appointment 12 (Progress)   → No form (< 15 visits)
Appointment 13 (Progress)   → No form (< 15 visits)
Appointment 14 (Progress)   → No form (< 15 visits)
Appointment 15 (Discharge)  → ✅ PROGRESS FORM (discharge trigger)
```

**Example 2: Surgical Patient (Pre-op/Post-op)**

```
Appointment 1 (Initial)     → No form (initial visit)
Appointment 2 (Progress)    → No form (< 5 visits)
Appointment 3 (Progress)    → No form (< 5 visits)
Appointment 4 (Progress)    → No form (< 5 visits)
Appointment 5 (Pre-op)      → ✅ PROGRESS FORM (cadence: 5th appointment)
[SURGERY DATE]
Appointment 6 (Re-eval)     → No form (counter reset, < 2 post-surgery)
Appointment 7 (Progress)    → No form (< 5 post-surgery visits)
Appointment 8 (Progress)    → No form (< 5 post-surgery visits)
Appointment 9 (Progress)    → No form (< 5 post-surgery visits)
Appointment 10 (Progress)   → No form (< 5 post-surgery visits)
Appointment 11 (Progress)   → ✅ PROGRESS FORM (cadence: 5th post-surgery)
```

**Note**: Pre-op forms are also created retrospectively if the patient had surgery and has post-op visits.

**Example 3: Early Discharge (< 5 visits)**

```
Appointment 1 (Initial)     → No form (initial visit)
Appointment 2 (Progress)    → No form (< 2 completed)
Appointment 3 (Discharge)   → ✅ PROGRESS FORM (discharge + ≥2 visits)
```

**Example 4: Very Early Discharge (< 2 visits)**

```
Appointment 1 (Initial)     → No form (initial visit)
Appointment 2 (Discharge)   → ❌ NO FORM (< 2 visits minimum)
```

**Example 5: Extended Treatment (20+ visits)**

```
Appointment 1 (Initial)     → No form (initial visit)
Appointment 2-4 (Progress)  → No form (< 5 visits)
Appointment 5 (Progress)    → ✅ PROGRESS FORM #1 (cadence: 5th)
Appointment 6-9 (Progress)  → No form (< 10 visits)
Appointment 10 (Progress)   → ✅ PROGRESS FORM #2 (cadence: 10th)
Appointment 11-14 (Progress) → No form (< 15 visits)
Appointment 15 (Progress)   → ✅ PROGRESS FORM #3 (cadence: 15th)
Appointment 16-19 (Progress) → No form (< 20 visits)
Appointment 20 (Progress)   → ✅ PROGRESS FORM #4 (cadence: 20th)
Appointment 21-24 (Progress) → No form (< 25 visits)
Appointment 25 (Discharge)  → ✅ PROGRESS FORM #5 (discharge trigger)
```

## Resumen de Reglas

### Cadence-Based Forms (Every 5th Visit)

- **5th appointment**: First cadence form
- **10th appointment**: Second cadence form  
- **15th appointment**: Third cadence form
- **20th appointment**: Fourth cadence form
- **25th appointment**: Fifth cadence form

### Special Trigger Forms

- **Discharge appointments**: Always create form if ≥2 visits completed
- **Pre-op appointments**: Created retrospectively if patient had surgery
- **Re-evaluation appointments**: Reset the visit counter

### Prerequisites for ANY Form Creation

1. ✅ At least 2 appointments completed
2. ✅ Intake form must be completed first
3. ✅ No existing progress form for that specific appointment
4. ✅ Haven't exceeded expected total forms for care plan

## Mathematical Formula

### Cadence Detection Logic

```python
is_cadence_collection = (effective_appointment_number + 1) % PROGRESS_FORM_EVERY_N_VISITS == 0

# With default setting of 5:
# Appointment 5: (4 + 1) % 5 = 0 ✅ 
# Appointment 10: (9 + 1) % 5 = 0 ✅
# Appointment 15: (14 + 1) % 5 = 0 ✅
# Appointment 20: (19 + 1) % 5 = 0 ✅
```

### Appointment Counter Rules

- **Initial visits**: Don't count toward progress form cadence
- **Re-evaluation visits**: Reset the effective counter to 0
- **Pre-op visits**: Reset counter after pre-op (post-surgery counting starts fresh)