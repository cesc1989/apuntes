# Script para crear Appointments para un Episode

Para poder hacer pruebas de todo tipo en local y/o alpha sin tener que ir a la app móvil.

Los posibles `visit_type` son:
```ruby
VISIT_TYPES = %w[initial reevaluation progress standard].freeze
```

Generalmente usaré solo `initial` y `standard`.

## Crear un solo Appointment

Para crear uno solo:
```ruby
appt = Appointment.new(
  episode: episode,
  region: episode.patient.region,
  visit_type: "initial_visit",
  state: :pending,
  scheduled_date: DateTime.new(2025, 6, 28, 15, 00, 00),
  scheduled_end_date: DateTime.new(2025, 6, 28, 15, 45, 00),
  location: episode.patient.locations.first,
  therapist: episode.patient.therapists.first,
  number: 1,
  created_by: episode.patient.therapists.first,
  booking_source: :unknown
)
appt.save
```

Tengo que tener en cuenta:

- buscar un Episode y asignarlo a `episode`
- Cambiar las fechas de `scheduled_date` y de `scheduled_end_date`
- `number` en 1 está bien pero tener en cuenta si el Episode tiene varios appointments

## Crear varios Appointments

```ruby
5.times do |i|
  visit_type = i == 0 ? "initial_visit" : "standard"
  scheduled_start = DateTime.new(2025, 6, 28, 15, 0, 0) + i.days
  scheduled_end = scheduled_start + 45.minutes

  appt = Appointment.new(
    episode: episode,
    region: episode.patient.region,
    visit_type: visit_type,
    state: :pending,
    scheduled_date: scheduled_start,
    scheduled_end_date: scheduled_end,
    location: episode.patient.locations.first,
    therapist: episode.patient.therapists.first,
    number: i + 1,
    created_by: episode.patient.therapists.first,
    booking_source: :unknown
  )

  unless appt.save
    puts "Error saving appointment ##{i + 1}: #{appt.errors.full_messages.join(', ')}"
  end
end
```

En este ya la clave es cambiar la fecha en `scheduled_start`.