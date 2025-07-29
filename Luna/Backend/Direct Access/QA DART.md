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



# Checklist Escenarios de Pruebas Generales

New Therapist Sign Up kicks workflow and webhook finds a 304 response because Therapist does not exist.
- [x] Contact is enrolled in the workflow
- [x] Goes through Active Attesting branch
- [x] Webhook responds with 304
- [x] Workflow is completed
- [x] No new `TherapistDirectAccessEntry` is created

Returning Therapist Sign Up kicks workflow and webhook creates a new `TherapistDirectAccessEntry` record.
- [x] Contact is enrolled in the workflow
- [x] Goes through Active Attesting branch
- [x] Webhook is successful
- [x] Workflow is completed
- [x] Creates a new `TherapistDirectAccessEntry` with label `active_attesting`.
- [ ] Changes previous `TherapistDirectAccessEntry` with label `active_attesting` to `active`

Updating Credentialing "Direct Access Restricted" property in HubSpot reflects to the `TherapistDirectAccessEntry` record

> [!Note]
> First the Therapist has to complete their Credentialing Application so that they can exist in Luxe db.

- [x] Contact is enrolled in the workflow
- [x] Goes through Active Attesting branch
- [x] Webhook is successful
- [x] Workflow is completed
- [x] `TherapistDirectAccessEntry` is created/updated


## Problemas Encontrados

Para el caso de Returning Therapists. Cuando se hace el 2do sign up, el Active Attesting initial permanece de esa forma. No hay nada que lo cambie.

Contact: https://app.hubspot.com/contacts/7712148/record/0-1/141932355901

```ruby
[#<TherapistDirectAccessEntry:0x00007f06bfe1af88
  id: "9b3113bd-271a-4825-9ea1-474d5fcd0f28",
  therapist_id: "ab7c3c6a-c7b1-4a5f-9605-595805f020b1",
  state_id: "d8853fcd-fd51-4e31-9030-fd9ae635cb55",
  permitted: true,
  hubspot_id: 31576122022,
  hubspot_created_at: Mon, 28 Jul 2025 18:39:11.000000000 PDT -07:00,
  association_label: "active_attesting",
  deleted_at: nil,
  created_at: Tue, 29 Jul 2025 09:08:11.299199000 PDT -07:00,
  updated_at: Tue, 29 Jul 2025 09:08:11.299199000 PDT -07:00>,
 #<TherapistDirectAccessEntry:0x00007f06bfdbbbc8
  id: "567f6710-756a-4b48-874e-d3399e60f563",
  therapist_id: "ab7c3c6a-c7b1-4a5f-9605-595805f020b1",
  state_id: "d8853fcd-fd51-4e31-9030-fd9ae635cb55",
  permitted: false,
  hubspot_id: 31644132664,
  hubspot_created_at: Tue, 29 Jul 2025 13:06:45.000000000 PDT -07:00,
  association_label: "active_attesting",
  deleted_at: nil,
  created_at: Tue, 29 Jul 2025 13:07:04.815998000 PDT -07:00,
  updated_at: Tue, 29 Jul 2025 13:07:04.815998000 PDT -07:00>]
```