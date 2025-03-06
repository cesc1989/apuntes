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

Di con esta forma de hacer una query para ver los datos de un paciente según lo que se define en el tipo.

### Query de patient (vieja forma)

```ruby
# app/graphql/types/query.rb
class Types::Query < Types::BaseObject
  field :patient, Types::Patient, null: false,
                                  authorize: true,
                                  deprecation_reason: "Redundant with `node`" do
    description "Look up a patient by ID."

    argument :id, ID, required: true do
      description "The ID of the patient."
    end
  end
end

# app/graphql/types/query.rb
class Types::Patient < Types::BaseObject
	field :first_name, GraphQL::Types::String, null: false do
    description "First name of this user."
  end

  field :care_plans, [Types::CarePlan], null: false do
  end
end
```

La query:
```json
{
  patient(id: "03da7897-7be3-4102-ac66-956ae61ac73d") {
    id,
    firstName,
    carePlans {
      id
    }
  }
}
```

La respuesta:
```json
{
  "data": {
    "patient": {
      "id": "03da7897-7be3-4102-ac66-956ae61ac73d",
      "firstName": "nippy wisteria akbash",
      "carePlans": [
        {
          "id": "ee51dc7f-ca06-4833-aaa6-4abf614fc59d"
        }
      ]
    }
  }
}
```

### Query usado Node

Tienen en Edge esta forma de hacer las queries a varios modelos:
```json
// el id es del CarePlan/Episode
{
  node(id: "ee51dc7f-ca06-4833-aaa6-4abf614fc59d") {
    ... on CarePlan {
      id,
      createdAt,
      name,
      payerPlan {
        id
      }
    }
  }
}
```

Responde:
```json
{
  "data": {
    "node": {
      "id": "ee51dc7f-ca06-4833-aaa6-4abf614fc59d",
      "createdAt": "2025-02-06T06:21:47-08:00",
      "name": "Ankle/Foot (04/05/2025 - 04/05/2025)",
      "payerPlan": {
        "id": "ea64e469-dbf1-4cd0-9ce0-4d2e17db978e"
      }
    }
  }
}
```


## Probando el campo `promptTherapistForMedicalNecessity`

Cuando pruebo la query de esta forma:
```json
{
  node(id: "ee51dc7f-ca06-4833-aaa6-4abf614fc59d") {
    ... on CarePlan {
      id,
      createdAt,
      name,
      payerPlan {
        id
      },
      promptTherapistForMedicalNecessity
    }
  }
}
```

Da este error:
```json
{
  "errors": [
    {
      "message": "undefined method `values' for #<BatchLoader::GraphQL:0x0000000118527100 @batch_loader=#<BatchLoader:0x1317560>>\n\n        context_key = context.values.map { _1._policy_cache_key(use_object_id: true) }.join(\".\")\n                             ^^^^^^^",
      "backtrace": [
        "/Users/francisco/.gem/ruby/3.1.6/gems/action_policy-0.6.0/lib/.rbnext/1995.next/action_policy/behaviours/policy_for.rb:62:in `policy_for_cache_key'",
        "/Users/francisco/.gem/ruby/3.1.6/gems/action_policy-0.6.0/lib/action_policy/behaviours/memoized.rb:37:in `__policy_memoize__'",
        ]
    }
  ]
}       
```

¿Qué lo causa?

## Ejecutar Mutaciones