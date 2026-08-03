# State Machine de `CareValidate::Request`

## Qué es

`CareValidate::Request` usa la gema [`state_machine`](https://github.com/state-machines/state_machines-activerecord) para controlar las transiciones de estado del modelo. La gema genera automáticamente métodos de instancia para consultar y cambiar el estado.

La state machine está definida en `app/models/care_validate/request.rb`.

## Estado inicial

`needs_requested_medpicker_data` — asignado automáticamente al crear un nuevo record.

## Estados

| Estado | Descripción |
|---|---|
| `needs_requested_medpicker_data` | Inicial. Esperando datos de MedPicker. |
| `pending` | Legacy — existe en el state machine pero sin transiciones activas. |
| `needs_prescriber_submission` | Datos de MedPicker listos, esperando envío a CareValidate. |
| `routed_to_beluga` | Derivada a Beluga Health en lugar de CareValidate. |
| `needs_crm_update` | Enviada al prescriptor, esperando sincronización con CRM. |
| `waiting_for_prescription` | CRM sincronizado, esperando webhook `ADD_CASE_DECISION` de CareValidate. |
| `needs_review` | Prescripción llegó pero requiere revisión manual. |
| `written` | Prescripción recibida y completa — estado final exitoso. |
| `needs_resubmission` | Revertida desde `written` para reenvío. |
| `canceled` | Cancelada — alcanzable desde cualquier estado. |
| `manually_resolved` | Resuelta manualmente sin seguir el flujo normal. |

## Eventos y transiciones

| Evento | Desde | Hacia |
|---|---|---|
| `ready_for_care_validate_submission` | `pending`, `needs_requested_medpicker_data` | `needs_prescriber_submission` |
| `ready_for_crm_sync` | `needs_prescriber_submission` | `needs_crm_update` |
| `waiting_for_prescription` | `needs_crm_update` | `waiting_for_prescription` |
| `received_prescription` | `waiting_for_prescription`, `needs_review` | `written` |
| `needs_review` | `waiting_for_prescription` | `needs_review` |
| `send_to_beluga` | `needs_requested_medpicker_data`, `needs_prescriber_submission`, `needs_crm_update` | `routed_to_beluga` |
| `manually_resolve` | `needs_requested_medpicker_data`, `waiting_for_prescription` | `manually_resolved` |
| `mark_needs_resubmission` | `written` | `needs_resubmission` |
| `reset_to_initial_state` | cualquier estado | `needs_requested_medpicker_data` |
| `cancel` | cualquier estado | `canceled` |

## Flujo principal (happy path)

```
needs_requested_medpicker_data   ← estado inicial
         │ ready_for_care_validate_submission!
         ▼
needs_prescriber_submission
         │ ready_for_crm_sync!
         ▼
needs_crm_update
         │ waiting_for_prescription!
         ▼
waiting_for_prescription
         │ received_prescription!
         ▼
written                          ← estado final exitoso
```

## Cómo se disparan las transiciones

La gema genera un método por cada evento definido. Hay dos variantes:

```ruby
request.ready_for_crm_sync    # retorna false si la transición no es válida, no lanza excepción
request.ready_for_crm_sync!   # lanza StateMachines::InvalidTransition si el estado actual no es válido
```

La versión con `!` (bang) también **persiste el cambio en la base de datos** automáticamente.

## No hay callbacks en el state machine

El bloque `state_machine` de `CareValidate::Request` **no define** `before_transition`, `after_transition`, ni ningún callback. Si los hubiera, se verían así:

```ruby
state_machine :state do
  after_transition on: :received_prescription do |request|
    # lógica post-transición
  end
end
```

Toda la lógica de negocio que ocurre antes o después de un cambio de estado vive en los **jobs y servicios que llaman al evento**, no en el modelo.

## Métodos de consulta generados

La gema también genera predicados de estado:

```ruby
request.needs_prescriber_submission?   # => true/false
request.written?                       # => true/false
request.can_ready_for_crm_sync?        # => true si la transición es válida desde el estado actual
```
