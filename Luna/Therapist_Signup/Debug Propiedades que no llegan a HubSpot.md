# Debuggeo de propiedades que no se llenan en HubSpot

## Contact Object

Contacto con el que reviso en HubSpot: https://app.hubspot.com/contacts/7712148/record/0-1/129069320278

### Preference

`treating_street_address`

- Sincronizó

## Credentialing Object - Active Attested

Credentialing Active Attested: https://app.hubspot.com/contacts/7712148/record/2-33642689/29453013680

### Immunization

`covid_19_vaccination_status`
`flu_vaccine_status`
`hepatitis_b_vaccination_status`
`mmr__measles__mumps__rubella__vaccination_status`
`tb_questionnaire__contact_with_individual_with_tb`
`tb_questionnaire__lived_in_countries_outside_the_us`
`tb_questionnaire__positive_tuberculosis_blood_test__quantiferon_gold_`
`tb_questionnaire__positive_tuberculosis_skin_test__ppd_`
`tb_questionnaire__received_bcg_vaccine`
`tb_questionnaire__taken_medication_for_tuberculosis_`
`tdap__tetanus__diphtheria__pertussis__vaccination_status`
`varicella__chickenpox__vaccination_status`

- Sincronizan

`tuberculosis_details` 🟡

- Está en código pero parece que no está en uso porque el campo no se usa

### Preference

`treatment_styles`
`bio_description`
`desired_weekly_appointments`
`do_you_have_any_pet_allergies_`
`languages`
`specialties_credentialing_form`

- Sincronizan

### NPI and CAQH

`caqh_number`
`caqh_password`
`caqh_permission_to_luna`
`caqh_username`

- No sincronizaba porque la app no tenía el scope `crm.objects.custom.highly_sensitive.write.v2`

`existing_caqh`
`existing_therapist_npi`
`therapist_npi_number`
`medicare_account_status`

- Sincroniza

### No está en código

`drivers_license_expiration_date` 🟡

`tb_questionnaire__current_tb_symptoms` 🟡
