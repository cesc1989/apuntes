# QA Plan para Local, Alpha y Omega - CD a Edge

Por si acaso. Todas las pruebas a correr:
```
pruebas spec/models/clinical_dashboard/ spec/lib/clinical_dashboard/ spec/services/clinical_dashboard/ spec/requests/clinical_dashboard/
```

## Local

As Admin user

- [x] Log in
- [x] Check data loads in visualizations
- [x] Filter by Physicians
- [x] Filter by Partners
- [x] Filter by Clinics
- [x] Filter by All Time

General

- [x] Rake task to request execution in Athena completes
- [x] Generate Physician Dashboard Link
- [x] Check Physician Dashboard link opens and loads visualizations
- [x] Check all filters
- [x] Check Unseen Patients section loads data from HubSpot
- [x] Sign an Open Task
- [x] Load a patient charts in the Recently Seen Patients table
- [x] Request Recently Seen Patients data download

Metrics

- [ ] Check all exports output CSV file to S3 folder

### Notes

Para poder probar Unseen Patients tengo que en HubSpot asociar Contactos de pacientes que:

- Hayan sido creados dos días atrás
- Tengan lead status diferente a "Patient - Qualified: Booked in Luxe"

## Alpha

As Admin user

- [x] Log in
- [x] Check data loads in visualizations
- [x] Filter by Physicians
- [x] Filter by Partners
- [x] Filter by Clinics
- [x] Filter by All Time

General

- [ ] Rake task to request execution in Athena completes
- [x] Generate Physician Dashboard Link
- [x] Check Physician Dashboard link opens and loads visualizations
- [x] Check all filters
- [x] Check Unseen Patients section loads data from HubSpot
- [x] Sign an Open Task
- [x] Load a patient charts in the Recently Seen Patients table
- [x] Request Recently Seen Patients data download

Metrics

- [ ] Check all exports output CSV file to S3 folder

Charts and Data Requests

- [x] Open Charts
- [ ] Download single chart
- [ ] Download charts in bulk
- [ ] Download all charts
- [ ] Requested patient data download arrives at email
- [ ] Exported patient data is correctly scoped

Open Tasks

- [ ] Reject a task
- [ ] Sign with comment
- [ ] Check back in Luxe/DB

Other

- [ ] Access CD after email verification completes
- [x] Submit referral

Dashboard Types

- [x] Generate Physician Dashboard
- [x] Generate Practice Dashboard
- [x] Generate Clinic Dashboard
