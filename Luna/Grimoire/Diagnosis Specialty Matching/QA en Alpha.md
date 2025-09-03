# QA en Alpha

Todos los casos que debo probar.

POW

- [x] outcome not booked && NTA syncs specialty.
	- [HS Contact](https://app.hubspot.com/contacts/7712148/record/0-1/151202252373)
	- [HS Contact 2](https://app.hubspot.com/contacts/7712148/record/0-1/150851711405)
- [x] outcome Patient placed on broadcast syncs specialty.
	- [HS Contact](https://app.hubspot.com/contacts/7712148/record/0-1/151093395784) 
	- [HS Contact 2](https://app.hubspot.com/contacts/7712148/record/0-1/151243277184)
- [x] outcome booked does not sync specialty.
	- [HS Contact](https://app.hubspot.com/contacts/7712148/record/0-1/151135734119)

CCW

- [x] outcome not booked && NTA syncs specialty.
	- [HS Contact](https://app.hubspot.com/contacts/7712148/record/0-1/149268630287)
- [x] outcome Patient placed on broadcast syncs specialty.
	- [HS Contact](https://app.hubspot.com/contacts/7712148/record/0-1/151063056007)
- [x] outcome booked does not sync specialty.
	- [HS Contact](https://app.hubspot.com/contacts/7712148/record/0-1/111037895816) 

## Todas las pruebas que debo correr

POW
```
mix test test/grimoire/patient_onboarding/saga_builder_test.exs
```

CCW
```
mix test test/grimoire/case_creation/saga_builder_test.exs
```

DiagnosisSpecialtyProperty
```
mix test test/grimoire/hubspot/diagnosis_specialty_contact_properties_test.exs
```

Todo junto:
```
mix test test/grimoire/patient_onboarding/saga_builder_test.exs test/grimoire/case_creation/saga_builder_test.exs test/grimoire/hubspot/diagnosis_specialty_contact_properties_test.exs
```