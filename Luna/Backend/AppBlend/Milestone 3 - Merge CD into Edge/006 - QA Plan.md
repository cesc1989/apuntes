# QA Plan para Local, Alpha y Omega - CD a Edg

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
- [ ] Request Recently Seen Patients data download

Metrics

- [ ] Check all exports output CSV file to S3 folder

## Notes

Para poder probar Unseen Patients tengo que en HubSpot asociar Contactos de pacientes que:

- Hayan sido creados dos días atrás
- Tengan lead status diferente a "Patient - Qualified: Booked in Luxe"