# Plan de QA para probar todo el sistema

Este plan contempla navegar y usar Luxe, probar integración entre servicios internos, completar ejercicios en la web, disparar background jobs y probar queries de GraphQL.

## Browse and interact in Luxe

- Actions Menu
	- [ ] Browse all sections
	- [ ] Create a deduction
- Patients / Profile
	- [ ] Search patient
	- [ ] Create care plan for patient
	- [ ] Check Document is attached after completing Intake Form
- Files menu
	- [ ] Open Document shares
		- [ ] Mark one as read
	- [ ] Visit Incoming faxes
- Clinical menu
	- [ ] Go to Auto charts section
	- [ ] Go to Plans of Care section
- Clinics / Profile
	- [ ] Add new portal recipient
	- [ ] Verify email for Clinical Dashboard
	- [ ] Send Clinical Dashboard email
- Appointments / Detail
	- [ ] Visit appointment detail
- Exercise Programs
	- [ ] Complete exercises in LE-W


## Trigger background jobs manually

Wilted Tree Reminder Worker
- [ ] Trigger worker
- [ ] Check wilted email was received

Clinical Dashboard Send Link Worker

Workers:
```
WiltedTreeExercisesViaWebReminderWorker.perform_async([1])

# Para practice Action On-Demand
ProviderPortalEmailSendingWorker.perform_async("618d2473-9d4c-4db4-9a91-741964a9d791")
```

## Test GraphQL queries
