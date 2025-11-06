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

bundle exec rspec spec/models/plan_of_care_spec.rb --fail-fast 2>/dev/null
```

# ¿Hay que dar soporte a business days?

En la petición la persona comenta que para Kansas:
> “Pending” at 10 **business** days. “Yes” at 15 **business** days or 10 visits

Luego de ver todo el código y entender lo que implica con Claude me quedó la duda si había que implementar lógica para _business days_. Claudio me confundió porque está instalada la gema `business_time`. Sin embargo, esa gema está instalada para temas de Candid.

Al revisar el commit donde se agrega y también al buscar código usando `business_days` solo aparecen cosas de Candid.

Adicional, le pregunté a Indy sobre esto y dijo:
> TX previous rule was business days so I don’t think it’s new to the system.

Y en otro comentario:
> Illinois is business days

Finalmente, le pedí un análisis a Claudio y dijo que:
> The business_time gem (~> 0.13.0) was installed to calculate business days and working hours for Luna's operations, specifically around:
>
> 1. **Patient Collections/Billing Operations** - Avoid sending communications on holidays
> 2. **Invoicing Runs** - Schedule automated billing runs only on supported business days (Tue-Fri)
> 3. **Working Date Calculations** - Determine next working day while respecting holidays

# Problema: POC de Progress Visit en Virginia 🟢

Indy reporta que:
> This VA pt just popped up on our list in “No violation”. For VA, the IV should be faxed but marked as response needed “NO” with no violation alerts. Also it looks like this was a PV faxed, only IVs should fax.

Hay dos cosas:
- For VA, the IV should be faxed but marked as response needed “NO” with no violation alerts.
- Also it looks like this was a PV faxed, only IVs should fax.

> [!Important]
> Descubrí junto con Claudio que el sistema está trabajando como debe. Se creó un 2do POC porque el Care Plan entró en proceso de recertificación.

Podría resumir el caso en que:

- El POC del Initial Visit fue actualizado a finales de Julio/2025
- A inicios de Octubre, por el proceso `DirectAccessPocsRecertificationWorker` se creó un nuevo POC
	- Este se creó porque se cumplieron los 75 días desde la última vez
	- En ese momento la visita más reciente fue una Progress

## Comprobar Care Plan entra en Recertificación

Con esta query puedo ver los POCs más sus actions y revisar si entra el primero en Recertificación.
```sql
SELECT
  poc.id AS poc_id,
  poc.created_at AS poc_created_at,
  poca.id AS action_id,
  poca.kind AS action_kind,
  poca.created_at AS action_created_at,
  poca.updated_at AS action_updated_at
FROM plans_of_care poc
LEFT JOIN plan_of_care_actions poca ON poca.plan_of_care_id = poc.id
WHERE poc.id IN ('POC_ID_INITIAL_VISIT', 'POC_ID_AFECTADO')
ORDER BY poc.created_at;
```

Y con esta puedo comprobar que un rango de fechas esté en los 75 días esperados:
```sql
SELECT
    '2025-10-11'::date - '2025-07-28'::date AS days_between_action_and_second_poc;
```

# Problema de POCs en Virginia 🐞

Reporte de dos partes.

**Primero: "response needed" debe ser ser "NO"**:
> This VA pt just popped up on our list in “No violation”.
> 
> For VA, ==the IV should be faxed but marked as response needed “NO” with no violation alerts==.

**Segundo: Progress Visits no deben generar fax**:
> Also it looks like this was a PV faxed, only IVs should fax.

Estos tres casos reportados el 16 de Octubre:
- Murphy
	- CP: `81ef64bc-303d-4ad8-aef2-956e510e1fa1`
	- POC ID: `4ee7ca7b-aa4e-4d5d-8b04-4d4f0c13ec3e`
		- POC creado el 14 de Octubre.
- Wiggins
	- POC ID: `52ef22c4-db12-4305-848c-54ab2a5bed8a`
		- POC creado el 14 de Octubre.
- Helm
	- POC ID: `5991ca00-9f88-47ff-9234-e021100d0caf`
		- POC creado el 13 de Octubre.

Y estos dos el 3 de Noviembre:
- Wright
	- Tiene dos CP activos pero solo aparece un POC.
	- POC ID: `f1740e94-718d-4ba8-b022-42812a5072a1`
		- POC creado el 27 de Octubre.
- Mittal
	- Tiene solo un CP activo. Solo un POC.
	- POC ID: `81c88226-2439-4a53-8ae2-29477da52777`
		- POC creado el 25 de Octubre.


## Murphy - PV: Recertification 🟢

Problema con Progress Visit (PV).

> [!Note]
> La PV recibió fax por proceso de recertificación.

Parece que es lo mismo de recertificación. Cuando busco los POCs del Care Plan encuentro tres. El más reciente fue creado el 14 de Octubre (dos días antes de que Indy comentara).

Al revisar a fondo puedo ver estos dos POCs:

- Initial Visit: `ebeb4864-8a16-49e3-8272-042abfc6d787`
	- creado el 31 de Julio
- Progress Visit: `4ee7ca7b-aa4e-4d5d-8b04-4d4f0c13ec3e`
	- creado el 14 de Octubre

Nota: también hubo un POC creado el 14 de Septiembre.

Cuando reviso la cantidad de días entre ambos da que son los 75 días para lo de recertificación.
```sql
SELECT
  poc.id,
  a.visit_type,
  poc.created_at,
  LAG(poc.created_at) OVER (ORDER BY poc.created_at) as previous_poc_created,
  poc.created_at - LAG(poc.created_at) OVER (ORDER BY poc.created_at) as days_between_pocs
FROM plans_of_care poc
JOIN charts c ON c.id = poc.chart_id
JOIN appointments a ON a.id = c.appointment_id
WHERE poc.id IN (
  'ebeb4864-8a16-49e3-8272-042abfc6d787',
  '4ee7ca7b-aa4e-4d5d-8b04-4d4f0c13ec3e'
)
ORDER BY poc.created_at;
```

Resultados:
```
days_between_pocs: 75
```

## Wiggins - IV 🟡

Problema con Initial Visit (IV). POC ID: `52ef22c4-db12-4305-848c-54ab2a5bed8a`

Para ver el problema toca revisar los audits para este POC. El detalle de este caso es que el POC fue creado con `needs_response` en "Yes":

| action | audited_changes |
|---------|-----------------|
| create | needs_response: true<br>response_received: false<br> |
| update | needs_response:<br>- true<br>- false<br>response_received:<br>- false<br>- true |

En el momento que fue creado el POC el campo `needs_response` era true. Luego fue actualizado manualmente por el equipo para reflejar el estado que esperan de estos POCs.

## Helm - IV 🟡

Problema con Initial Visit (IV). POC ID: `5991ca00-9f88-47ff-9234-e021100d0caf`.

POC fue creado con `needs_response` en "Yes".

## Wright - IV 🟡

Problema con Initial Visit (IV). POC ID: `f1740e94-718d-4ba8-b022-42812a5072a1`.

POC fue creado con `needs_response` en "Yes".

## Mittal - IV 🟡

Problema con Initial Visit (IV). POC ID: `81c88226-2439-4a53-8ae2-29477da52777`.

POC fue creado con `needs_response` en "Yes".

## Problema

El problema ocurre porque al crear el POC siempre se mira si el appointment está `discharged` para darle el valor a `needs_response`. Sin embargo, para los Initial Visits esto se cumple más bien poco.

Comparativa de Initial Visit con estado discharged en false:

- Wiggins
	- discharged: false
- Helm
	- discharged: false
- Wright
	- discharged: false
- Mittal
	- discharged: false