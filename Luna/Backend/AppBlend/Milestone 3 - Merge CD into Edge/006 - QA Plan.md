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

- [x] Check all exports output CSV file to S3 folder

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

- [x] Rake task to request execution in Athena completes
- [x] Job to request execution in Athena completes
- [x] Generate Physician Dashboard Link
- [x] Check Physician Dashboard link opens and loads visualizations
- [x] Check all filters
- [x] Check Unseen Patients section loads data from HubSpot
- [x] Sign an Open Task
- [x] Load a patient charts in the Recently Seen Patients table
- [x] Request Recently Seen Patients data download
- [x] Request New Link page is functional - [Example Page](https://provider-portal.alpha.getluna.com/dashboard/eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwaHlzaWNpYW4iLCJleHAiOjE3NTE0ODA2NTgsImlhdCI6MTc1MTQ4MDE1OC44NjE2MTIsImlzcyI6InBoeXNpY2lhbi1wb3J0YWwtbGlua3MtZ2VuZXJhdG9yIiwibmJmIjoxNzUxNDgwMTU4LCJzdWIiOiI2ZjFhMTMwYi1kNDk5LTQ2ZTEtYWE4Yi0xYjU1MmE0MTMwMTIiLCJwcm92aWRlcl9uYW1lIjoiSHVnaCBUaGVQaHlzaWNpYW4iLCJwcm92aWRlcl9raW5kIjoicGh5c2ljaWFuIiwicHJvdmlkZXJfaWQiOiI2ZjFhMTMwYi1kNDk5LTQ2ZTEtYWE4Yi0xYjU1MmE0MTMwMTIiLCJwb3J0YWxfcmVjaXBpZW50X2VtYWlsIjoiZnJhbmNpc2NvLnF1aW50ZXJvKzNAaWRlYXdhcmUuY28iLCJkYXNoYm9hcmRfdmVyc2lvbiI6InYzIiwiZGFzaGJvYXJkX2lkIjoiNDUyYjVmM2EtNWMzZC00M2RlLTg3ZWMtMTI5YTcyNWNiNjQ4In0.0OYWw_UU9GiLjrGzZIPmXrnEb5jmV5MK76lR_nQoJhA)

Metrics

- [ ] Check all exports output CSV file to S3 folder

Charts and Data Requests

- [x] Open Charts
- [x] Download single chart
- [x] Download charts in bulk
- [x] Download all charts
- [x] Requested patient data download arrives at email
- [x] Exported patient data is correctly scoped

Open Tasks

- [x] Reject a task
- [x] Sign with comment
- [x] Check back in Luxe/DB

Other

- [x] Access CD after email verification completes
- [x] Submit referral

Dashboard Types

- [x] Generate Physician Dashboard
- [x] Generate Practice Dashboard
- [x] Generate Clinic Dashboard
