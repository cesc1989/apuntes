# Luna Patients Forms June 14th Release
This document is an overview for the release scheduled to be deployed at **08:00 AM California time, 10:00 AM Colombia time**.

## Accountable People
- Francisco Quintero
- Carlos Lobo
- Darwin Wu

In case of errors, Francisco and Carlos would debug and try to deliver a quick fix.
In case of bad and complicated errors, **suggested action is to rollback deployment**.

**Migration rollback command**


    $ rails db:rollback STEP=4

Migrations that would be rolled back, newer to older:


- AddPersistDraftUntilToForm
- AddsExcludeMedicalInformationToForms
- AddsReferredCarePhysicianNameToIntakeForms
- AddCarePhysicianReferralSourceToIntakeForms
## Introduced Changes

Backend:


- Release pull request is https://github.com/lunacare/patient-forms-backend/pull/154
- **147 commits** to be merged from January 29th, 2019
- Actual test coverage is **94.7%**
- All changes integrated in frontend forms and tested in staging environment

Frontend:


- Release pull request is https://github.com/lunacare/patient-forms-frontend/pull/138
- **61 commits** to be merged from January 24th, 2019
- All changes tested in staging environment
    - Tested intake forms and progress forms in **Chrome, Firefox, Brave, and Safari**. Also on mobile
    - Tested new implementation of drafts/in-progress intake and progress forms
    - Tested new implementation of loading data of previous intake for 2nd case a patient requires new treatment

**Backend Changes**
In summary:


- Show corresponding IntakeForm and ProgressForm date in Summary
- Add Sentry to project(this is already on master branch)
- Add care physician referral source, referral therapist to IntakeForm model
- Add exclude medical information to Form model
- Introduce RequestAdmissionForm form object
- [DevOps] Add multi env config
- [DevOps] Add staging config file
- Pre-assign patient attributes on AdmissionForm
- Refactor to acceptance of terms and conditions
- Refactor to presence of patient's signature
- Add and configure simplecov for test coverage
- Configure rubocop rules
- Add editorconfig rules
- Implement Intake form drafts
- Implement Progress form drafts
- Submit intake/progress form based on a draft
- Implement previous intake endpoint
- Add Application Load Balancer rules for new drafts and previous intake endpoints
- Add patient's signature attribute to update patient endpoint

**Frontend Changes**
In summary:


- Rewrite table functionality to adjust better on responsive sizes
- Fix date format on surgery table of summary view
- Increase pain spots size for mobile
- Add referral Source feature to intake form
- Add alert when accessing to form with a mobile browser (iOS or android)
- Skip terms and conditions for existing patient
- Exclude medical information if flag is sent from API
- Prevent users to edit Agg. Activities in progress forms
- Allow users to add Agg. Activities in progress forms if no data is present.
- Add Date/Month pickers for mobile browsers
- Fix typo on a functional limitation answer option in the LEFS form
- Update terms and conditions on 2nd page
- Remove grey lines at the bottom of the form
- Update terms and conditions page design
- Save in-progress work for onboarding forms
- Save in-progress work for progress forms
- Fix expired link handling. It was throwing an error.
- Fix weird red border for last date of exam input on Firefox.

