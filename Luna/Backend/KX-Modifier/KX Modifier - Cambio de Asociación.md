# KX Modifier - Cambio de Asociación entre Episode y MDTS

Resulta que toca cambiar la relación porque un Episode podrá tener Appointments en dos años diferentes.

## En Episode

Estos son los cambios en el modelo Episode:
```diff
+ has_one :latest_medicare_dollar_threshold_status
+      -> { effective_per_episode.order(effective_from: :desc) },
+      class_name: "MedicareDollarThresholdStatus"

- has_one :medicare_dollar_threshold_status
+ has_many :medicare_dollar_threshold_statuses

- has_one :medicare_care_plan_medical_necessity_response, foreign_key: "care_plan_id"
+ has_many :medicare_care_plan_medical_necessity_responses, foreign_key: "care_plan_id"
```

El controlador en ActiveAdmin estaba preparado para recibir una colección ya que en el modelo Patient y MDTS se define la asociación con las macros para nested attributes.

Patient:
```ruby
has_many :medicare_dollar_threshold_statuses
accepts_nested_attributes_for :medicare_dollar_threshold_statuses
```

MDTS:
```ruby
has_many :medicare_care_plan_medical_necessity_responses
accepts_nested_attributes_for :medicare_care_plan_medical_necessity_responses, allow_destroy: true
```

Por eso lado, la vista funciona correctamente.

¿Por qué cambia a has_many Medical Necessity?

Por este caso:

- Un Care Plan que pasa el límite en Diciembre 2025.
- Su MDTS queda marcado como excedido.
- Llegamos a 2026 y el MDTS se desmarca.
- Seguirá teniendo el MDTS del año 2025


## En MedicareCarePlanMedicalNecessityResponse

Ahora vamos a permitir varias respuestas para un mismo Care Plan, acotado a un solo MDTS.

Este es el cambio:
```diff
- validates :care_plan,
-    uniqueness: { message: "Response already submitted" },
-    presence: { message: "Care plan is required" }
+ validates :care_plan, uniqueness: { scope: :medicare_dollar_threshold_status, message: "Response already submitted" }
```

Lo que significa que ahora un mismo Care Plan podrá tener:

- Varios MDTS (solo uno por año)
- Varias `necessity_responses` por cada MDTS


## Sobre el scope `effective_per_appointment`

En el modelo MTDS, Alexis agregó este scope:
```ruby
scope :effective_per_appointment, lambda {
    medicare_dollar_threshold_statuses_for_appointments =
      Appointment
      .select("applicable_medicare_dollar_threshold_statuses.*")
      .joins(<<-SQL)
        JOIN LATERAL
          (SELECT medicare_dollar_threshold_statuses.*, appointments.id AS appointment_id
            FROM medicare_dollar_threshold_statuses
            JOIN patients ON patients.id = medicare_dollar_threshold_statuses.patient_id
            JOIN episodes ON episodes.patient_id = patients.id
            JOIN regions ON regions.id = appointments.region_id
            WHERE appointments.episode_id = episodes.id
              AND MAKE_DATE(
                EXTRACT(
                  YEAR FROM TIMEZONE('UTC', appointments.scheduled_date) AT TIME ZONE regions.time_zone
                )::integer, 1, 1
              ) = medicare_dollar_threshold_statuses.effective_from
            AND
            MAKE_DATE(
              EXTRACT(
                YEAR FROM TIMEZONE('UTC', appointments.scheduled_date) AT TIME ZONE regions.time_zone
              )::integer, 12, 31
            ) = medicare_dollar_threshold_statuses.effective_until
          ) AS applicable_medicare_dollar_threshold_statuses ON TRUE
      SQL

    from("(#{medicare_dollar_threshold_statuses_for_appointments.to_sql}) medicare_dollar_threshold_statuses")
  }
```

Y así lo usa en el modelo Appointment:
```ruby
has_one :medicare_dollar_threshold_status, -> { effective_per_appointment }, class_name: "MedicareDollarThresholdStatus"
```

> [!Note]
> Me llama la atención el uso del scope del modelo MDTS en la macro definida en otro modelo.
> 
> Creía que eso podría estar mal pero en el mismo Appointment hay otra asociación que sigue el mismo patrón: `has_one :effective_bond, -> { effective_per_appointment }, class_name: "TherapistPatientBond"` y en el modelo `TherapistPatientBond` se define el scope.
>
> Vale aclarar que la tabla `therapist_patient_bonds` no tiene `appointment_id`

El uso del scope en la definición de la asociación sirve porque Rails lo que está haciendo es aplicar el scope como si fuera un `where`.