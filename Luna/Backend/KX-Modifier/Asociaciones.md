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