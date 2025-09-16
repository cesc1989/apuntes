# Episode (Care Plan) Discharge Process

Etiquetas: #luna_help_desk

This document explains how episodes (care plans) change to discharged status.

## Episode Status Enum

Episodes have a `status` enum with these values:
- `active: 0` - Care plan is ongoing
- `auto_discharged: 1` - Automatically discharged by the system
- `treatment_completed: 2` - Manually discharged (treatment finished)
- `draft: 666` - Draft status

## Discharge Methods

### 1. Manual Discharge (`treatment_completed`)

Episodes are manually discharged using the `CarePlan::DischargeForm`.

**Triggering conditions:**
- A therapist or admin manually discharges the care plan
- Requires a `treatment_completed_reason` (goals_met, md_asked_to_discontinue, etc.)
- Can include a `treatment_completed_note`

**What happens:**
- Status changes to `treatment_completed`
- `discharged_at` timestamp is set
- `scheduling_enabled` is set to `false`
- Future appointments are automatically canceled
- Final chart is updated with discharge flag
- Patient and therapist notifications are sent

### 2. Auto Discharge (`auto_discharged`)

Episodes are automatically discharged by the `EpisodeAutoDischargeWorker`.

**Triggering conditions:**
- Episode is `active` status
- Last appointment was before the auto-discharge period (default: 30 days ago based on `auto_discharge_period_in_seconds` setting)
- Patient is not on an active waitlist
- OR: Care plan was created before auto-discharge period but has no appointments

**What happens:**
- Status changes to `auto_discharged`
- `discharged_at` timestamp is set
- `scheduling_enabled` is set to `false`
- Auto-discharge event notifications are published

## Key Code Locations

- **Episode Model**: `app/models/episode.rb`
  - Status enum definition: line 500
  - Discharge scope: line 501 (`scope :discharged`)
  - Status change callback: `Episode#set_discharged_at` (line 1777)

- **Manual Discharge Form**: `app/forms/care_plan/discharge_form.rb`
  - Main discharge method: `CarePlan::DischargeForm#discharge!`
  - Auto discharge method: `CarePlan::DischargeForm#auto_discharge!`

- **Auto Discharge Worker**: `app/workers/episode_auto_discharge_worker.rb`
  - Automatic discharge logic: `EpisodeAutoDischargeWorker#perform`

## Treatment Completed Reasons

When manually discharging, one of these reasons must be selected:

- `goals_met` - Goals met
- `md_asked_to_discontinue` - MD asked to discontinue
- `insurance_moving_away` - Insurance / moving away
- `moving_to_clinic_location` - Moving to clinic location
- `no_signed_poc_direct_access` - No signed POC / Direct Access
- `deterioriation_returned_to_md` - Deterioration / return to MD
- `inappropriate_patient` - Inappropriate patient
- `requires_higher_level_of_care` - Requires higher level of care
- `poor_patient_experience` - Poor patient experience
- `unable_to_connect_for_further_scheduling` - Unable to connect for further scheduling
- `plateau_in_progress_or_not_needed` - Plateau in progress / No longer needs treatment

## Database Changes on Discharge

When an episode is discharged, the following fields are updated:

- `status` → `auto_discharged` (1) or `treatment_completed` (2)
- `discharged_at` → Current timestamp
- `scheduling_enabled` → `false`
- `treatment_completed_reason` → Reason enum value (for manual discharge)
- `treatment_completed_note` → Optional note (for manual discharge)

## Related Scopes and Methods

- `Episode.discharged` - Returns both auto_discharged and treatment_completed episodes
- `Episode#discharged?` - Returns true if not active and not draft
- `Episode.recent_discharge` - Episodes discharged within last 90 days
- `Episode#days_since_first_completed_visit` - Helper for calculating care duration

## Summary

Episodes become discharged either through:
1. **Manual intervention** (treatment completed by therapist/admin)
2. **Automatic system processes** (auto-discharged after 30+ days of inactivity)

Both result in the care plan being marked as finished, scheduling being disabled, and appropriate notifications being sent.