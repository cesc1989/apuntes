# Enums más comunes en Edge

Etiquetas: #luna_help_desk 

Esta una lista de enums de estado de entidades en Edge/Luxe. Que funcione como documento madre para así hallar más fácil estos estados y los posibles valores. Teniendo en cuenta que la mayoría se guardan como enteros en la base de datos.

## Episode

Episodes have a `status` enum with these values:
- `active: 0` - Care plan is ongoing
- `auto_discharged: 1` - Automatically discharged by the system
- `treatment_completed: 2` - Manually discharged (treatment finished)
- `draft: 666` - Draft status

Definición:
```ruby
enum status: {
  active: 0,
  auto_discharged: 1,
  treatment_completed: 2,
  draft: 666
}
```

## Appointment

Los posibles `visit_type` de Appointment son:
```ruby
VISIT_TYPES = %w[initial reevaluation progress standard].freeze
```

Enum `state` de Appointment:
```ruby
enum state: {
	pending: 0,
	ongoing: 1,
	completed: 2,
	canceled: 3,
	no_show: 5,
	# Unpersisted
	draft_add: 6,
	draft_reschedule: 7,
	draft_cancel: 8
}
```

Estados draft e inactivo:
```ruby
DRAFT_STATES = %w[draft_add draft_reschedule draft_cancel]
INACTIVE_STATES = %w[canceled no_show].freeze
```

Estados activo:
```ruby
ACTIVE_STATES = %w[pending ongoing completed].freeze
```

## Chart

Enum `state` de Chart:
```ruby
enum state: {
	pending_for_audio: 0,
	pending_for_review: 1,
	pending_for_approval: 2,
	signed: 3,
	rejected_by_reviewer: 5,
	rejected_by_therapist: 6,
	pending_for_pdf: 7,
	waiting_for_call: 8,
	edited: 9,
	resubmitted_by_therapist: 10
}
```

## PlanOfCareAction

Enum `kind` de PlanOfCareAction:
```ruby
enum kind: {
	resolved_by_admin: 0,
	signed_by_physician: 1,
	physician_requested_modifications: 2,
	rejected_by_physician: 3,
	physician_requested_modifications_handled_by_admin: 4,
	rejected_by_physician_handled_by_admin: 5
}
```

## PayerAuthorization

Enum `request_status` en PayerAuthorization:
```ruby
enum request_status: {
	pending_authorization: 0,
	granted: 1,
	denied: 2
}
```

## Availability

Enum `creation_source` en Availability:
```ruby
enum creation_source: {
	luxe: 1,
	mobile_patient_app: 2,
	mobile_therapist_app: 3,
	omni: 10,
	unknown: 666
}
```