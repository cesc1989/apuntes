# KX Modifier - Cambio de Asociación entre Episode y MDTS

Resulta que toca cambiar la relación porque un Episode podrá tener Appointments en dos años diferentes.

## scope effective_per_appointment

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
> Me llama la atención el uso del scope del modelo MDTS en la macro definida en otro modelo. Creía que eso podría estar mal pero en el mismo Appointment hay otra asociación que sigue el mismo patrón: `has_one :effective_bond, -> { effective_per_appointment }, class_name: "TherapistPatientBond"` y en el modelo `TherapistPatientBond` se define el scope.
> .
> Vale aclarar que la tabla `therapist_patient_bonds` no tiene `appointment_id`