# Propiedades que no se llenan en HubSpot

Tomadas del reporte donde varias propiedades no son actualizadas según lo que se llene en el formulario CF o AF.

# Prueba en Alpha

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

# Prueba en Omega

Contact: jose+newuat1111@invoketin.com

```
"credentialing_active_attested_id": 29373649988,
"credentialing_hubspot_id": 29370588936,
"credentialing_inactive_hubspot_id": 29355517616,
"credentialing_relocation_id": 29389557604,

"hubspot_id": 128577747727,
"license_hubspot_id": 29367319847,
"therapist_address_hubspot_id": 29386258081,
```

El test case para "New Therapist" apunta al Credentialing con ID `29355517616`. Este es Inactive.

## Notas de propiedades

La propiedad de Contact `treating_street_address` reportada como no sincronizada en el archivo creo que indica a lo que sería "second street line".

Para el caso de Contact no existe para Treatment second street line.

Para el caso de Therapist Address sí existe. Está marcada como que sí sincroniza.

---

Hay un error al sincronizar las propiedades para Preference:
```
"errors":[{"message":"Property \"therapist_requests_support_on_documentation_and_cpt_codes_for_outpatient_billing\" does not exist","code":"PROPERTY_DOESNT_EXIST"
```

La propiedad `therapist_requests_support_on_documentation_and_cpt_codes_for_outpatient_billing` no está en la cuenta de Omega y da error. Puede ser que por eso fallaron algunos syncs.

> [!Tip]
> Resuelto. La propiedad fue archivada pero ahora la activaron de nuevo.

---

Hay otro error al intentar sincronizar al Credentialing object.

```
{"status":"error","message":"This app hasn't been granted all required scopes to make this call. Read more about required scopes here: https://developers.hubspot.com/scopes.","correlationId":"c757f599-2ce3-42f0-bf5d-18304bfab4b4","errors":[{"message":"One or more of the following scopes are required.","context":{"requiredGranularScopes":["crm.objects.custom.highly_sensitive.write.v2"]}}],"links":{"scopes":"https://developers.hubspot.com/scopes"},"category":"MISSING_SCOPES"}
```

Hace falta el scope `crm.objects.custom.highly_sensitive.write` en la private app de Omega.

---

