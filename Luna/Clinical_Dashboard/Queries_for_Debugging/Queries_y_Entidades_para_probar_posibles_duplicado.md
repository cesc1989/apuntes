# Queries y Entidades para probar posibles duplicados al actualizar el PatientSummaryWriter de Luxe

Estas entidades y queries me ayudarán a revisar que no pasen casos como

[+🐞 Duplicated patients because of multiple escalations [Portal]](https://paper.dropbox.com/doc/Duplicated-patients-because-of-multiple-escalations-Portal-oJdaNpCK7tZu5vimZmOd6)

[+[Report] Fix for duplication of patients because of Treatment Completed Reason](https://paper.dropbox.com/doc/Report-Fix-for-duplication-of-patients-because-of-Treatment-Completed-Reason-i8A8twbwnYnxDngIFk8yM) 


# Query para hacer debug de datos de Paciente en Athena

Esta query me va a servir para traer la información de Patients Treated, revisarla en Athena o exportarla a excel sin tener que andar ocultando columnas.

```sql
/*Patients_Treated Debugging Query*/
SELECT
  CONCAT(pat.first_name, ' ', pat.last_name) AS patient_name,
  pat.id AS patient_id,
  CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
  phy.id AS physician_id,
  forms.form_type,
  part.name AS partner_name,
  part.id AS partner_id,
  pat.earliest_visit_date AS initial_visit_date,
  pat.latest_visit_date AS latest_visit_date,
  cli.id AS clinic_id,
  pat.escalated,
  pat.escalated_at,
  pat.escalated_reason,
  pat.escalation_id,
  pat.discharged,
  pat.treatment_completed_reason
FROM "application-data"."patients" pat
LEFT JOIN "application-data"."patient-forms" forms ON pat.id = forms.internal_id
LEFT JOIN "application-data"."physicians" phy ON phy.id = pat.physician_id
LEFT JOIN "application-data"."practices" part ON part.id = pat.practice_id
LEFT JOIN "application-data"."clinics" cli ON cli.id = pat.clinic_id
WHERE (
  (pat.completed_visits_count = 0 AND pat.pending_visits_count > 0)
  OR
  (pat.completed_visits_count > 0 AND pat.pending_visits_count >= 0)
)
AND (
  pat.latest_visit_date IS NULL
  OR pat.latest_visit_date = ''
  OR pat.latest_visit_date > to_iso8601(current_date - interval '90' day)
)
ORDER BY pat.last_name;
```


# Alpha: IDs y Providers para Pruebas

Physician

- Nombre: Aaron Fuancho
- ID: `some-uuid`

Clinic

- Nombre: Luna Care San Francisco
- ID: `other-uuid`


## Pacientes que permiten ver el caso de duplicación de treatment_completed_reason
| Paciente           | ID                                   | Clinical Dashboard               |
| ------------------ | ------------------------------------ | -------------------------------- |
| Paompos O'Hara     | some-uuid | Dr. Fuancho                  |
| Patient P01T5      | e7d55167-3eac-4523-a2cf-0678ea0a70dd | Luna Care San Francisco (Clinic) |
| Patient Ty opt - 1 | d72aa222-f73e-4add-a145-8919b2b55e39 | Luna Care San Francisco (Clinic) |
