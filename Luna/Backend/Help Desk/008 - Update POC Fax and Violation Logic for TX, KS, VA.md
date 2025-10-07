# 008 - Update POC Fax and Violation Logic for TX, KS, VA

Etiquetas: #luna_help_desk 

Caso EDG-2746

Reporte:
> Update POC fax and violation logic for 3 direct access states.
>
> New laws are in effect in 3 states and we need POC logic to match
>
> TX, KS, and VA all had law updates for direct access. Due to these changes, we need to update faxing logic and violation logic

# Contexto

Los diferentes estados en los que Luna opera se configuran mediante seeds en `db/seeds/states`. Esta carpeta contiene archivos yml con información de cada estado. Las llaves de nivel superior son:

- `name`
- `postal_abbreviation`
- `plan_of_care_rules_config`
- `regions`
- `stripe_accounts`

En la llave `plan_of_care_rules_config` se configura la llave `direct_access` que es la que contiene las que interesan a este cambio:
- `fax_rules`
- `violation_rules`

Es en estas que tengo que hacer las modificaciones que piden.

YAML de ejemplo:
```yaml
---
name: Kansas
postal_abbreviation: KS
plan_of_care_rules_config:
  direct_access:
    fax_rules:
      anchor: initial_visit
      trigger:
        strategy: chart_signature
    violation_rules:
      pending:
        days_elapsed: 10
      violation:
        days_elapsed: 21
    resolution_mode: functional_progress
regions: []
stripe_accounts: []
```

Hay una clase seeder que se encarga de leer todos estos archivos y prepara todo para actualizar los registros de la tabla. El seeder está en `app/seeders/state_seeder.rb`.

> [!Note]
> La rake para correr estos seeds es `bundle exec rails db:seed`

## ¿Qué son Fax Rules y Violation Rules

Según Claudio:

- POC faxing logic: When POCs are faxed to physicians
- Violation logic: When compliance is notified of violations

Estas reglas son usadas en las clases `PlansOfCare::DirectAccess::ViolationCalculator` para Violation Logic y en `PlansOfCare::DirectAccess::MessageCalculator`  para Fax Rules.

### Fax Rules

Este se rige por los valores de la llave `fax_rules`. A la fecha, toda configuración que tiene esta llave tendrá esta pareja:
```
anchor: initial_visit
```

Para la llave `trigger` los posibles valores para `strategy` son:

1. `chart_signature`: Fax immediately when chart is signed
2. `days_elapsed`: Fax after N days from anchor
3. `days_or_visits_elapsed`: Fax after N days OR M visits (whichever comes first)

> [!Tip]
> Cuando hay `strategy: chart_signature` no suele haber la key `days` por lo que se explica antes.

### Violation Rules

Se rige por los valores de la llave `violation_rules`. Cuando existe puede tener las llaves anidadas `pending` y `violation`.

> [!Tip]
> El equipo de Luna suele referirse a `pending` como "pending" y a `violation` como "yes".


# Solución

Los primeros cambios son actualizar los archivos yml de Texas, Virgina y Kansas.

Para esta petición no hay necesidad de modificar `PlansOfCare::DirectAccess::MessageCalculator` ya que soporta los valores de las actualizaciones.

Sin embargo, `PlansOfCare::DirectAccess::ViolationCalculator` sí necesita actualización para soportar _business days_.

## Verificación de Actualización de los Fax Rules

Con estos comandos en la rails console se puede verificar:
```ruby
# Check TX configuration
tx = State.tx
tx.plan_of_care_rules_config["direct_access"]["fax_rules"]["trigger"]["days"]
# Should return 15

tx.plan_of_care_rules_config["direct_access"]["violation_rules"]["pending"]["days_elapsed"]
# Should return 20

tx.plan_of_care_rules_config["direct_access"]["violation_rules"]["violation"]["days_elapsed"]
# Should return 30

# Check KS configuration
ks = State.ks
ks.plan_of_care_rules_config["direct_access"]["violation_rules"]["violation"]["days_elapsed"]
# Should return 15

ks.plan_of_care_rules_config["direct_access"]["violation_rules"]["violation"]["visits_elapsed"]
# Should return 10

ks.plan_of_care_rules_config["direct_access"]["violation_rules"]["violation"]["day_type"]
# Should return "business"

# Check VA configuration
va = State.va
va.plan_of_care_rules_config["direct_access"]["fax_rules"]["trigger"]["strategy"]
# Should return "chart_signature"

va.plan_of_care_rules_config["direct_access"]["violation_rules"]
# Should be nil or absent
```

## Pruebas automáticas

Las pruebas a correr para verificar en local:
```bash
bundle exec rspec spec/services/plans_of_care/direct_access/ --fail-fast 2>/dev/null
```

```bash
bundle exec rspec spec/models/plan_of_care_spec.rb --fail-fast 2>/dev/null
bundle exec rspec spec/models/state_spec.rb --fail-fast 2>/dev/null
```