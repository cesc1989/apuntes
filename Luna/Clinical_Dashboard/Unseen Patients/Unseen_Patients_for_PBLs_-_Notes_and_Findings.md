# Unseen Patients for PBLs - Notes and Findings

PBL = Powered by Luna Partners = Clinics, Practices.

# Hubspot Properties

**Powered by Luna (PBL) Partner Code**

- Hubspot property: *powered_by_luna_code*
- Example: BLR

**PBL Company Name**

- Hubspot property: *pbl_company_name*
- Example: Luna Physical Therapy

**First Touch Lead Source**

- Hubspot property: *lead_source*
- Example: Powered by Luna

# Athena tables and fields

**Clinics table**

- key
    - example: luna_care_philadelphia
- provider_name
    - example: Luna Care, Inc

**Practices table**

- name
    - example: DVACO
- code
    - example: DVA

# Matching Patients to Practices

Made a query to list patients and practices with code DVA and HOP, went to Hubspot and looked for contacts with same last names and found ten occurrences for each practice code.

# Matching Patients to Clinics

There’s no a single property to single out a patient linked to a clinic. We would have to instead, if the clinic is linked to a practice, use the practice’s code to find the patient in Hubspot.

# Other Notes

Multiple patients appear in a Practice and a Clinic dashboards. Some examples include *Marion A Mignogna, Gulnara A Acosta*.

Some patients are linked to clinics but not to practices. Usually, these patients would not show up in Hubspot. Example of these are *Jalal Aboudi*, *Lori G Abramowitz*.

Multiple clinics are not linked to Practices. Usually, these clinics are named Luna Care, Inc.

# Backend Tasks

- Actualizar query `PROD_Normalized_Partners_All_Time` para que devuelva el `code`.
    - Actualiza la query para que use un prefijo `prat` en vez de `cli`.
- Actualizar query `PROD_Normalized_Clinics_List_All_Time` para que devuelva el `code`.
    - Hay que hacer left join con practices para conseguir el código.
- Actualizar el converter `PartnersConverter` para que devuelva el código del practice a la lista de selección.
- Actualizar el converter `ClinicsConverter` para que devuelva el código del practice a la lista de selección.
- Setear datos en Alpha hubspot para poder probar.
- Usar el `code` enviado por frontend para hacer la búsqueda en Hubspot.
    - Aquí ya no es con asociación sino buscando en una propiedad: ¿qué endpoint sería?

# Frontend Tasks
- Enviar el `code` en las peticiones para filtrar los datos.

