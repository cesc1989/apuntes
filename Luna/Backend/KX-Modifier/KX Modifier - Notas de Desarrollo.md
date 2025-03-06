# Notas de Desarrollo de KX Modifier

Todo lo que voy descubriendo de Edge a medida que hago esta vaina.

# ¿Cuándo un Episode es Medicare?

En Episode veo esto:

```ruby
class Episode < ApplicationRecord
  # Types of care plan referrals
  scope :medicare,
        -> { joins(:insurance).where(insurances: { key: %w[medicare medicare_advantage] }) } # insurance is Medicare


  # (...)

  # "Straight Medicare" i.e. coverage directly provided by the federal government
  # Centers for Medicare & Medicaid Services (CMS).
  # Not to be confused with commercial payer plans i.e. Medicare Advantage
  def medicare?
    is_medicare_plan_category = payer_plan&.category&.medicare? == true

    # XXX: Remove after new plan categories are live
    is_medicare_insurance = insurance&.medicare? == true
    is_medicare_payer = payer&.name&.starts_with?("Medicare") == true && payer_plan&.category&.government? == true

    is_medicare_plan_category || is_medicare_insurance || is_medicare_payer
  end
end
```

> [!Note]
> Context
> Alexis le preguntó a Christie: "is this strictly for medicare insurances or does it include medicare advantage?"
> 
> Ella contestó: "Medicare only"

Creo que tocaría hacer un filtro como este:
```ruby
medicare_care_plans = patient.episodes.select(&:medicare?)
```

O mejor un scope nuevo que quite la opción `medicare_advantage`:
```ruby
scope :straight_medicare, -> { joins(:insurance).where(insurances: { key: "medicare" }) }
```

¿Tiene sentido algo así? Creo que sí. Cuando reviso en alpha por posibles Insurances que sean `medicare_x` encuentro tres:
```ruby
Insurance.where("key LIKE ?", "%medicare%").pluck(:id, :key)

[[3, "medicare"], [28, "medicare_advantage"], [396, "medicare_secondary"]]
```

## Insurance que es Medicare

En el modelo Insurance veo estos métodos:
```ruby
class Insurance < ApplicationRecord
  def medicare?
    key == "medicare"
  end

  def medicaid?
    key == "medicaid"
  end

  def medicare_advantage?
    key == "medicare_advantage"
  end
end
```

# Estados de Appointment

## Estado Activo

Los appointments están activos cuando están en alguno de estos estados:
```ruby
ACTIVE_STATES = %w[pending ongoing completed].freeze
```

# Usando GraphiQL

Se accede desde `http://localhost:3000/graphiql`. Hay que primero iniciar sesión como admin en Luxe.

## Ejecutar Queries



## Ejecutar Mutaciones