# KX Modifier Pruebas en Alpha: Luxe y GQL

Entidades y notas sobre las pruebas en Alpha de la integración entre Luxe y GQL.

## Activar / Desactivar Feature Flag

```ruby
Flipper.enable(:kx_modifier)

Flipper.disable(:kx_modifier)
```

## Entidades

Necesito:

- Care plan *medicare* con un solo appointment
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