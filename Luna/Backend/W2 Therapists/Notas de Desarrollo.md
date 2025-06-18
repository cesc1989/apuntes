# Notas de Desarrollo - W2 Therapists

# Relación entre Therapists y Patients

La asociación es `therapist -> patients` pero tiene detalles:
```ruby
class Therapist < ApplicationRecord
  # All patients this therapist is allowed to view in the app.
  has_many :patients, lambda { |therapist|
    visible_patients = Patient.union(
      Patient.visible_via_schedule_for(therapist),
      Patient.visible_via_discharge_for(therapist),
      Patient.visible_via_bond_for(therapist)
    )
    Patient.where(id: visible_patients).unscope(where: :therapist_id)
  }, class_name: "Patient"
end
```

Veamos esos scopes en `Patient`:
```ruby
class Patient < ApplicationRecord
  scope :visible_via_schedule_for, lambda { |therapist|
    where(
      Episode
      .active
      .where("patients.id = episodes.patient_id")
      .where(
        Appointment
        .active
        .where(therapist_id: therapist.id)
        .where("episodes.id = appointments.episode_id")
        .select(1).arel.exists
      )
      .select(1).arel.exists
    )
  }
  
  scope :visible_via_discharge_for, lambda { |therapist|
    where(
      Episode
      .discharged
      .recent_discharge
      .where("episodes.patient_id = patients.id")
      .where(last_treating_therapist: therapist)
      .select(1).arel.exists
    )
  }
  
  scope :visible_via_bond_for, lambda { |therapist|
    where(
      TherapistPatientBond
      .where("therapist_patient_bonds.patient_id = patients.id")
      .where(therapist_id: therapist.id)
      .select(1).arel.exists
    )
  }
end
```

Las claves son:

- Para `visible_via_schedule_for` que son patients con care plan (episode) `active`
- Para `visible_via_discharge_for` que son patients con care plan `discharged`
	- _Ojo aquí: esto es el scope `discharged`_
- Para `visible_via_bond_for` son patient donde la relación `TherapistPatientBond` con therapist existe

# Estados de Therapist

Son estos:
```ruby
enum status: %i[pending_activation activated paused moved not_interested not_a_fit doa expired_license draft]
```

Por la sintaxis los estados están dados por el índice del array. O sea que los valores enteros serían:
```ruby
pending_activation: 0
activated: 1
paused: 2
moved: 3
not_interested: 4
not_a_fit: 5
doa: 6
expired_license: 7
draft: 8
```

# Estados de Episode

Estos son los diferentes estados de un Episode:
```ruby
enum status: {
  active: 0,
  auto_discharged: 1,
  treatment_completed: 2,
  draft: 666
}
```

## 🟡 Atento con el scope `discharged` 🟡

En muchas partes se usa este scope. No representa ningún estado sino una combinación>
```ruby
scope :discharged, -> { where(status: %i[auto_discharged treatment_completed]) }
```

Es `discharged` cuando:

- su estado cambió a `auto_discharged`
- su estado cambió a `treatment_completed`

# Estados de Appointment

Voy a necesitar tener esto claro para poder completar las validaciones.

> [!Note]
> Por defecto, los appointments son creados con `state = 0`

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

También están los estado activo:
```ruby
ACTIVE_STATES = %w[pending ongoing completed].freeze
```

Y los estado inactivo:
```ruby
INACTIVE_STATES = %w[canceled no_show].freeze
```

## Scopes

Hay scopes para filtrar los appointments según su estado:
```ruby
scope :active, -> { where(state: ACTIVE_STATES) }
scope :inactive, -> { where(state: INACTIVE_STATES) }
```

# Script para probar servicio en Therapist Sign Up

De momento, se necesita que se puedan crear therapists en Therapist Sign Up sin para por el form. Para eso Alexis hizo un servicio que se puede invocar pasando los parámetros que se envían desde el sign up form.

Así lo pruebo en local:
```ruby
params = {
  therapist: {
    first_name: "Juan 004",
    last_name: "Pérez 004",
    desired_signature: "JP",
    phone_number: "1234567890",
    email: "juan.perez+004@example.com",
    treating_postal_code: "12345",
    entity_kind: "Individual",
    physical_therapy_license_number: "PT12345",
    physical_therapy_license_expiration_date: "2025-12-31",
    government_photo_id: "ID12345",
    back_of_government_photo_id: "ID12345-back",
    government_id_expiration_date: "2025-12-31",
    basic_life_support_certificate: "BLS12345",
    basic_life_support_certificate_number: "BLS12345",
    basic_life_support_expiration_date: "2025-12-31",
    basic_life_support_issuing_company: "American Heart Association",
    accepted_terms_and_conditions: true,
    registered_from: "california",
    care_start_date: "2025-10-29",
    resume: "hola"
  }
}

service = EmployeeTherapistSignUpFormService.new(params)
service.create
```