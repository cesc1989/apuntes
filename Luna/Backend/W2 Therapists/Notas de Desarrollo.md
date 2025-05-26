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

