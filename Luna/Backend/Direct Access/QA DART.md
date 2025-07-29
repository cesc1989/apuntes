# QA DART - Todas las Pruebas

Hay dos cosas a tener en cuenta:

- Escenarios de pruebas iniciales
- Escenarios de agendamiento (booking)

## Escenarios de Pruebas Generales

Lo que tiene que ver con crear un registro en `therapist_direct_access_entries` y que se pueda actualizar dicho registro.

- Nuevo Sign Up da error cuando el Therapist aún no existe
	- El Contacto se enrola en el workflow
	- Ingresa a la rama Active Attesting
	- Falla el webhook
	- Se completa el workflow
- Cuando se completa el Credentialing Form????
- Segundo Sign Up crea un nuevo registro
	- El Contacto se enrola en el workflow
	- Ingresa a la rama Active Attesting
	- Se completa el webhook
	- Se completa el workflow
- Actualizar propiedades selectas del Credentialing actualiza el registro
	- El Contacto se enrola en el workflow
	- Ingresa a la rama Active Attesting
	- Se completa el webhook
	- Se completa el workflow


## Escenarios de Agendamiento (Booking)

### Happy paths

**Booking**

1. When the **patient has** a medical referral on their care plan
    1. And the therapist is a **DART** in the patient’s state
        1. Then the **PT can** be successfully booked with the patient
    2. And the therapist is a **DAPT** in the patient’s state
        1. Then the **PT can** be successfully booked with the patient
2. When the **patient does not have** a medical referral on their care plan
    1. And the therapist is a **DART** in the patient’s state
        1. Then the **PT cannot** be booked with the patient _This is true for Medicare patients, as well. Calling this out because pre-Direct Access project, Medicare patients w/o medical referrals were occasionally booked with DARTs on the optimistic assumption the patient would have a medical referral by the time of the IV. This should be explicitly disallowed now._
    2. And the therapist is a **DAPT** in the patient’s state
        1. Then the **PT can** be successfully booked with the patient

**Broadcast**

1. When the **patient has** a medical referral on their care plan
    1. And the patient is waitlisted
        1. And the therapist is a **DART** in the patient’s state
            1. Then the PT **can view** this patient on their Broadcast
            2. And the **PT can** successfully book the patient
        2. And the therapist is a **DAPT** in the patient’s state
            1. Then the PT **can view** this patient on their Broadcast
            2. And the **PT can** successfully book the patient
2. If the **patient does not have** a medical referral on their care plan
    1. And the patient is waitlisted
        1. And the therapist is a **DART** in the patient’s state
            1. Then the **PT cannot** **view** this patient on their Broadcast
            2. And the **PT cannot** successfully book the patient
        2. And the therapist is a **DAPT** in the patient’s state
            1. Then the PT **can view** this patient on their Broadcast
            2. And the **PT can** successfully book the patient

### Edge cases

1. If the **patient does not have** a medical referral on their care plan
    1. And the therapist is a **DAPT** in the patient’s state
        1. And the same therapist is a **DART** in another state that is not the patient’s
            1. The **PT can** treat the patient, and **can view** if the patient is on Broadcast
    2. And the therapist is a **DART** in the patient’s state
        1. And the same therapist is a **DAPT** in another state that is not the patient’s
            1. The **PT cannot** treat the patient, and **cannot view** if the patient is on Broadcast



# Checklist

New Therapist Sign Up kicks workflow and webhook finds a 304 response because Therapist does not exist.
- [ ] Contact is enrolled in the workflow
- [ ] Goes through Active Attesting branch
- [ ] Webhook responds with 304
- [ ] Workflow is completed
- [ ] No new `TherapistDirectAccessEntry` is created

Returning Therapist Sign Up kicks workflow and webhook creates a new `TherapistDirectAccessEntry` record.
- [ ] Contact is enrolled in the workflow
- [ ] Goes through Active Attesting branch
- [ ] Webhook is successful
- [ ] Workflow is completed
- [ ] A new `TherapistDirectAccessEntry` is created

Updating Credentialing properties in HubSpot reflects to the `TherapistDirectAccessEntry` record
- [ ] Contact is enrolled in the workflow
- [ ] Goes through Active Attesting branch
- [ ] Webhook is successful
- [ ] Workflow is completed
- [ ] `TherapistDirectAccessEntry` is updated