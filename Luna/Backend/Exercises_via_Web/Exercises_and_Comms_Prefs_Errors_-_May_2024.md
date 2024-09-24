# ✅ Exercises and Comms Prefs Errors - May/2024

# FIXED: No Comms Token crashes EvW website

LogRocket recordings: [go to page](https://app.logrocket.com/ps0ymv/luna-care/metrics/209974?from=latest&imIssueEvent=d2c4c02cddaac779553d5638dd67839c&imIssueFilters=%255B%257B%2522networkErrorUrl%2522%253A%257B%2522operator%2522%253A%2522LIKE%2522%252C%2522strings%2522%253A%255B%2522api2.getluna.com%252Fapi%252Fv1%252Fexternal%252Fexercise_programs%252F%2A%2522%255D%257D%257D%252C%257B%2522networkErrorStatus%2522%253A%257B%2522operator%2522%253A%2522EQ%2522%252C%2522strings%2522%253A%255B%2522500%2522%255D%257D%257D%252C%257B%2522networkErrorMethod%2522%253A%257B%2522operator%2522%253A%2522IS%2522%252C%2522strings%2522%253A%255B%2522GET%2522%255D%257D%257D%252C%257B%2522issuePlatformType%2522%253A%257B%2522operator%2522%253A%2522IS%2522%252C%2522strings%2522%253A%255B%2522Web%2522%255D%257D%257D%255D&imIssueID=i&imIssueType=NETWORK_ERROR&imTab=sessions).

Page: https://api2.getluna.com/evw/patients/8Z5NMgQJT5wnz-d8sUZN/exercise_programs/29700abf-721b-4768-8450-8338f8b530d4

Sentry error: [https://sentry.omega.getluna.com/share/issue/2ef0693c7ebc4e84882a800698a53b47/](https://sentry.omega.getluna.com/share/issue/2ef0693c7ebc4e84882a800698a53b47/)

Error:

    NoMethodError
    undefined method `token' for nil:NilClass
    
    web_token = patient.patient_communication_preference_token.token

Patient lacking a `patient_communication_preference_token`.

# No Settings Found for Patient in Comms Pref

Error: Some patients are not having their `patient_communication_preference_token` created. Investigate different places where patient creation happen.

Also, there’s no logic in place to generate `notification_settings` for new patient Accounts.

Tests:

- In alpha, identify patients without `patient_communication_preference_token`. 
- In alpha, identify patient accounts without `notification_settings`.

Fixes:

- Rake task to generate missing comms_token
- Rake task to generate missing notification_settings for patient accounts
- Logic to create notification_settings for new patient accounts

