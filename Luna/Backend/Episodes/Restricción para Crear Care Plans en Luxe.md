# Care Plan Creation Restriction - Summary

**Date**: 2025-12-05
**Environment**: Luna Backend (Alpha & Omega)
**Issue**: Cannot create first care plan for new patients in Luxe admin interface

---

## Root Cause

### Ticket: EDG-1848 - "Shut off onboarding features now in Bliss"

**Date**: February 2025
**Author**: Jeff Brateman
**Assignee**: Luis Marroquin

**Purpose**: Move patient onboarding from Luxe (Backend) to Bliss system

**Requirements**:
1. Shut off the ability to create new patients
2. **Shut off the ability to create an initial care plan (patient's first care plan)**
3. Leave everything else untouched

---

## Implementation Details

### Location

`app/admin/care_plans.rb` - Lines 863-869

### Code Flow

**Original Implementation** (Commit 38a1a6b6e - PR #11356):
```ruby
unless Episode.exists?(patient_id: params[:patient_id]) || current_admin_user.super_admin?
  redirect_to(admin_care_plans_path) && return
end
```
- Blocks first care plan creation for regular admins
- **Allows super admins to create first care plans**

**Current Implementation** (Commit 18d7d4c18 - "Fix cloning even more"):
```ruby
if Episode.where(patient_id: params[:patient_id]).empty? && params[:episode_id].blank?
  redirect_to(admin_care_plans_path) && return
end
```
- Blocks first care plan creation for ALL admins
- **Removed super admin exception** (likely unintentional)

---

## Behavior

### When Creating a New Care Plan

The `new` action checks:
1. Is `patient_id` present? (No → redirect)
2. Does patient have existing care plans? (No → redirect)
3. Is this a cloning operation? (No → redirect)

**Result**: New patients without existing care plans **cannot** get their first care plan created through Luxe admin, regardless of admin permissions.

### Button Visibility

The "Create New Care Plan" button also has a separate check in the patient view:

**Location**: `app/admin/customers/patients.rb` - Lines 1567-1571
```ruby
if Episode.exists?(patient_id: patient.id) || current_admin_user.super_admin?
  render partial: "add_careplan_button", locals: {
    patient: patient,
    recent_draft_care_plan: patient.recent_draft_care_plan
  }
end
```

**Note**: The button check STILL has the super admin exception, but the `new` action does not!

---

## Workarounds

### Option 1: Create First Care Plan via Console (Alpha)

```ruby
patient = Patient.find("patient-uuid")
specialty = Specialty.find_by(key: "ORTHOPEDICS_AND_SPORT_MEDICINE")

Episode.create!(
  patient: patient,
  specialty: specialty,
  status: "draft",
  region: patient.region
)
```

After creating this first care plan, subsequent care plans can be created through the admin UI (cloning or normal creation).

### Option 2: Use Bliss for Patient Onboarding

Create patients and their first care plan through the Bliss system (as intended).

---

## Related Files

- `app/admin/care_plans.rb` (lines 863-869) - New action redirect logic
- `app/admin/customers/patients.rb` (lines 1567-1571) - Button visibility check
- `app/views/admin/patients/_add_careplan_button.html.erb` - Button template

---

## Related Pull Requests

1. **PR #11356** - "Remove Create Care Plan Option from Admin Views"
   - Commit: 38a1a6b6e
   - Author: Luis Marroquin
   - Date: Feb 28, 2025

2. **"Fix cloning even more"**
   - Commit: 18d7d4c18
   - Author: Ryan Gaffney
   - Date: Feb 28, 2025
   - **Note**: This commit removed the super admin exception

---

## Business Context

**Why This Restriction Exists**:
- Patient onboarding (including first care plan creation) has been moved to the Bliss system
- Luxe admin interface is now primarily for managing existing care plans
- This prevents duplicate/conflicting onboarding workflows

**Impact**:
- New patients created outside Bliss cannot get their first care plan through Luxe
- Super admins lost the ability to override this restriction (likely unintentional)
- Subsequent care plans (after the first) can still be created normally
