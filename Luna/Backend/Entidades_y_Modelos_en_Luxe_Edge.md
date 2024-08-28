# Entidades y Modelos en Luxe/Edge

Hay mucha nomenclatura y términos que desconozco o no entiendo de Luxe. En este documento los anoto según voy viendo/entendiendo para no estar tan en la ignorancia.

- Episode == Care Plan

![[luxe-portal-export.png]]

## Appointment

- belongs_to Therapist
- belongs_to Episode
- has_one Patient through Episode


## Therapist

- has_many Appointments
- has_many Patients through Appointments


## Episode

- belongs_to Patient
- has_many Appointments
- has_many Therapist through Appointments
- belongs_to Physician


## Patient

- has_many Episodes
- has_many Appointments through Episodes


## Physician

- has_many Episodes

# Entidades

- Physician
- Patient
- Physician Group
    - *Physician Groups are groups of physicians which are bundled together in the Portal*
- PBL = Powered By Luna
    - *marketing term for a clinic group or hospital or other partner*
- Partner Clinic = Practice
    - *Physical therapy practice, such as Luna Care or Spooner PT*
- Clinic

