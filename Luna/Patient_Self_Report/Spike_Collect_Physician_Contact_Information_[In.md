# ✅ Spike: Collect Physician Contact Information [Intake Form]

# About
> Add a **prompt** after the physician name to collect the physician’s email address and phone number on the Intake Form.
![](https://paper-attachments.dropboxusercontent.com/s_ED418798010B085E72C870C4AA30794314EBB9A065D0D8DBFCC8570EDD2AEE9B_1708373740430_image.png)


**Example Intake Form**
https://forms.alpha.getluna.com/patients/abe5047d-492d-4a5d-b9a9-8a1a5b0372ca/forms/d2f72d83-563b-408b-93b6-fa1720388158

## Questions and Details
- Are there any prerequisites?
- Sync back to Hubspot
- Pull physician information when creating Intake Form
- Is physician information synced to Hubspot from Luxe?
- How much effort is this work?
# Are there any prerequisites?

Yes, there are.

First, add two new fields to the `intake_forms` table:

- `physician_email` (string)
- `physician_phone` (string)

Second, merge these fields into Omega.

> Q: Why?
> A: Because there’s a restriction to merge DB changes to Alpha environment. We have to first have these changes in Omega if we want to try these feature set in Alpha.
# Sync back to Hubspot

Currently, no information about the physician is being synced back to Hubspot.

With this feature coming in, we can start syncing physician data to Hubspot by finding or creating the physician as Hubspot Contact using the provided email address.

Use these properties to save this new details to the Hubspot contact:

- Name of care physician → `firstname`
- Physician Email → `email`
- Physician Phone → `phone`
# Pull physician information when creating Intake Form

Marketplace does not save physician’s phone number or email address. There’s no current association between Care Plan and Physician in this backend.

To pull these values I’d need to fetch the info from Luxe via an endpoint. This endpoint would be called in a background job after the Form is created.

However, this might not be as straight forward because, as Ryan explains:

> The patient likely isn’t associated w a physician by the time they fill out their intake form
# Is physician information synced to Hubspot from Luxe?

Yes. Physician records have a `hubspot_id` attribute. This indicates syncing back and forth to Hubspot.

In the `Physician#resync_from_hubspot` function, physician info in Luxe is updated with info from Hubspot.

In the `HubspotSyncPhysicianWorker` execution, hubspot properties `physician_patient_count` and `physician_care_plan_count` are updated based on current values from Luxe.

> done every 5 minutes

In `HubspotSyncReferralWorker` job, physicians are created/updated in Hubspot from their referred information.


> About referrals: They’re created in the `Api::V1::Therapist::ReferralsController`. After successfully saved, the job `HubspotSyncReferralWorker`, in the `Referral` model, is started.

In the controller `PhysicianController`, physician data is update from a Hubspot webhook.

# How much effort is this work?

With the prerequisites in place, what’s left to do:

1. Whitelist the new fields in the endpoint to save form draft and submit the form.
2. Add the new input fields in the “Current Medical History” section.
3. QA form draft and submission with the new fields.
    1. If doing sync to Hubspot, test this flow in Luna Alpha account in Hubspot.



