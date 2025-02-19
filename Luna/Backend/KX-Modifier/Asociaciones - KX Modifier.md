# Notas de las Asociaciones para KX Modifier

## has_one :through has_many

En el modelo Episode se define esto:
```ruby
has_one :medicare_dollar_threshold_status,
  -> { where(effective_from: initial_visit.scheduled_date.all_year) },
  through: :patient
```

Pero daba este error:
```bash
Failure/Error: expect(episode.medicare_dollar_threshold_status).to be_a(MedicareDollarThresholdStatus)

     ActiveRecord::HasManyThroughSourceAssociationNotFoundError:
       Could not find the source association(s) "medicare_dollar_threshold_status" or :medicare_dollar_threshold_status in model Patient. Try 'has_many :medicare_dollar_threshold_status, :through => :patient, :source => <name>'.
```

Para arreglarlo tocó usar la opción `:source`:
```ruby
has_one :medicare_dollar_threshold_status,
  -> { where(effective_from: initial_visit.scheduled_date.all_year) },
  through: :patient,
  source: :medicare_dollar_threshold_statuses
```

Para que Rails pueda inferir el nombre de la asociación para encontrar los registros.

## Leer instancia dentro del ámbito de asociación

Esta línea:
```ruby
-> { where(effective_from: initial_visit.scheduled_date.all_year) },
```

causa este error:
```ruby
Failure/Error: -> { where(effective_from: initial_visit.scheduled_date.all_year) }, NameError: undefined local variable or method `initial_visit' for #<ActiveRecord::Relation []>` Did you mean? initialize
```

Eso porque, en palabras de ChatGPT:
> El problema es que estás intentando acceder a `initial_visit.scheduled_date` dentro del scope de la lambda, pero en ese contexto, `initial_visit` no está disponible porque la lambda se ejecuta en el contexto de la relación y no en el contexto de la instancia del modelo.

Se arregla así la relación:
```ruby
has_one :medicare_dollar_threshold_status,
  ->(record) { where(effective_from: record.initial_visit.scheduled_date.all_year) },
  through: :patient,
  source: :medicare_dollar_threshold_statuses
```

donde `record` representa la instancia del modelo actual.

## has_one through usando instancia en scope

Tengo estas relaciones:
```ruby
# Appointment
has_one :medicare_dollar_threshold_status,
        through: :episode

# Episode
has_one :medicare_dollar_threshold_status,
	lambda { |episode|
		where(
			effective_from: episode.initial_visit.scheduled_date.beginning_of_year,
			effective_until: episode.initial_visit.scheduled_date.end_of_year
		)
	},
	through: :patient,
	source: :medicare_dollar_threshold_statuses
```

Cuando quiero correr una prueba con estas asociaciones da este error:
```bash
     NoMethodError:
       undefined method `initial_visit' for #<Appointment id
```

Pasa que Rails está tratando de llamar `initial_visit` en una instancia de Appointment en lugar de Episode.

Esto es porque en
```ruby
appt.medicare_dollar_threshold_status
```

Rails pasa `appt` en el contexto de la lambda definida en `medicare_dollar_threshold_status` en el modelo Episode. En vez de hacer algo como:

```
appointment -> episode -> medicare_dollar_threshold_status
```

Hace algo como:
```
appointment -> medicare_dollar_threshold_status
```

Y esto último no nos sirve.