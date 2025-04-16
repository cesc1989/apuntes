# KX Modifier Pruebas en Alpha: Luxe y GQL

Entidades y notas sobre las pruebas en Alpha de la integración entre Luxe y GQL.

## Activar / Desactivar Feature Flag

```ruby
Flipper.enable(:kx_modifier)

Flipper.disable(:kx_modifier)
```

## Entidades

Necesito:

- Care plan *medicare* con al menos un appointment
- Su respectivo paciente
- El therapist de ese appointment

En variables para luego poder leer los respectivos IDs y crear los registros necesarios.

Query para encontrar care plans con esa característica:
```ruby
epi = Episode.straight_medicare
            .joins(:appointments)
            .group('episodes.id')
            .having('COUNT(appointments.id) = 1')
            .first
```

Comandos para probar los valores:
```ruby
ap epi.id
ap epi.medicare?
ap "Cantidad de citas: #{epi.appointments.count}"
ap "Initial Visit ID: #{epi.initial_visit.id}"
ap "Initial Visit Date: #{epi.initial_visit.scheduled_date}"
ap "Initial vist therapist ID: #{epi.initial_visit.therapist.id}"
ap epi.medicare_dollar_threshold_status
```

Al actualizar el `scheduled_date` se debe crear el MedicareDollarThresholdStatus:
```ruby
epi.initial_visit.update(scheduled_date: epi.initial_visit.scheduled_date + 1.day)
```

Para marcar el `threshold_exceeded` como true:
```ruby
epi.medicare_dollar_threshold_status.update(threshold_exceeded: true)
```

Para crear el Medical Necessity Response:
```ruby
MedicareCarePlanMedicalNecessityResponse.create(
  medicare_dollar_threshold_status: epi.medicare_dollar_threshold_status,
  care_plan: epi,
  therapist: epi.initial_visit.therapist,
  medical_necessity_state: :approved
)
```

## Crear Initial Visit

Es el 1er appointment. Como esto suele crearse desde la app o desde la interfaz rara que no conozco, lo haré directo por base de datos.

```ruby
epi = Episode.find("3a1da366-cce9-4344-84ca-ffc6a9b232f9")

appt = Appointment.new(
  episode: epi,
  region: patient.region,
  visit_type: "initial_visit",
  state: :pending,
  scheduled_date: DateTime.new(2025, 4, 21, 15, 00, 00),
  scheduled_end_date: DateTime.new(2025, 4, 21, 15, 45, 00),
  location: patient.locations.first,
  therapist: epi.patient.therapists.first,
  number: 1,
  created_by: epi.patient.therapists.first,
  booking_source: :unknown
)
ap appt.valid?
ap appt.errors.messages
appt.save
```

## Crear Chart para Appointment

Si el appointment no tiene chart, puedo crear uno así de fácil:
```ruby
appt.create_chart
```

Después le cambié el estado para que sea firmable:
```ruby
appt.chart.update(state: 2)
```

O para que esté firmada:
```ruby
appt.chart.update(state: 3)
```

Recuerda generar auth para el Therapist para hacer las peticiones mediante Postman:
```ruby
print Account.find("steve+nt@getluna.com").create_new_auth_token
```

Y hacer el proceso en la terminal para obtener las creds como cabeceras HTTP para usar en Postman:
```bash
$ json='hash_copiado_del_comando_anterior'

$ headers_from_ruby_hash "$json"
```

# Pruebas

## Perfil de Care Plan / Perfil Paciente, panel Care Plan

Si no hay `medicare_dollar_threshold_status`:

- [x] No muestra nada.

Si hay `medicare_dollar_threshold_status` con `threshold_exceeded` en false:

- [x] Muestra la info sobre "Spending Limit Exceeded".

Si hay `medicare_dollar_threshold_status` y `threshold_exceeded` en true:

- [x] Muestra la info sobre "Spending Limit Exceeded".
- [x] Muestra mensaje "Prompt Therapist".

Si tiene `MedicareCarePlanMedicalNecessityResponse`, se muestra info de si la necessity fue aprobada o rechazada:

- [x] Muestra la info sobre "Spending Limit Exceeded"
- [x] Muestra info sobre "Medical Necessity Response".
- [x] Muestra info sobre "Medical Necessity Response From".


## Form de Editar Paciente

Si no hay `medicare_dollar_threshold_status`:

- [x] No muestra nada.
- [x] Guarda el form al editar campos de Paciente.

Si hay `medicare_dollar_threshold_status` con `threshold_exceeded` en false:

- [x] Muestra la info de "MedicareDollarThresholdStatus" junto ID de Care Plan.
- [x] Guarda el form al editar campos de Paciente.
- [x] Guarda el form al editar campos de "MedicareDollarThresholdStatus".

Si tiene `MedicareCarePlanMedicalNecessityResponse`, se muestra info sobre si la necessity fue aprobada o rechazada:

- [x] Muestra los campos de Medical Necessity.
- [x] Guarda el form al editar campos de Paciente.
- [x] Guarda el form al editar campos de Medical Necessity.


# Otras Entidades

## Credenciales de Therapist para acceder desde la App

Hay que cambiarle la clave para iniciar sesión:
```ruby
ap "PT email: #{epi.initial_visit.therapist.account.email}"

epi.initial_visit.therapist.account.update_attribute(:password, "clave-super-segura")
```