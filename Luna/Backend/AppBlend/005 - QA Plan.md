# Plan de QA para probar todo el sistema

Este plan contempla navegar y usar Luxe, probar integración entre servicios internos, completar ejercicios en la web, disparar background jobs y probar queries de GraphQL.

- Browsed an interacted in several pages in Luxe
    - Actions Menu
        - Create a deduction
    - Patients / Profile
        - Search patient
        - Create care plan for patient
        - Document attached after completing Intake Form
    - Files menu
        - Document shares
            - Mark one as read
        - Incoming faxes
    - Auto charts
    - Clinics profile
    - Appointments / detail
    - Exercise Programs
        - Complete exercises in LE-W
    - Verify email for clinical dashboard
    - Send clinical dashboard email
-  Trigger some background jobs manually
    - Wilted Tree Reminder Worker
    - Clinical Dashboard Send Link Worker
- GraphQL: ¿cómo?

Workers:

```
WiltedTreeExercisesViaWebReminderWorker.perform_async([1])

# de practice Action On-Demand
ProviderPortalEmailSendingWorker.perform_async("618d2473-9d4c-4db4-9a91-741964a9d791")
```