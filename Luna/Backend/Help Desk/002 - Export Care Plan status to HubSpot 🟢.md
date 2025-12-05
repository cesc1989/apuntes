# 002 - Export Care Plan Status to HubSpot

Etiquetas: #luna_help_desk

Tengo que exportar los valores de Care Plan active y `scheduling_enabled` a las propiedades `active_care_plan` y `care_plan_scheduling_enabled` del contacto del paciente.

Las dos propiedades las cree como "Single checkbox" así que esperan un valor true o false.

## Pruebas Manuales

### Active y `scheduling_enabled` en true

Así encontré un Care Plan que cumple ambas condiciones:
```ruby
eps = Episode
 .joins(patient: :account)
 .where(status: :active, scheduling_enabled: true)
 .where.not(account: { hubspot_id: nil })

fep = eps.first

ap fep.patient.account.hubspot_id
ap fep.active?
ap fep.most_recent_care_plan?
```

Para dispara el worker cambié el `phone_number`:
```ruby
fep.update(phone_number: "3013972802")
```

Pasan un montón de cosas así que no pude registrar en el log el worker que me interesa pero pude comprobar en HubSpot que ambas propiedades se actualizan.

Contacto de prueba: https://app.hubspot.com/contacts/7712148/record/0-1/140618664050

### Care Plan es `discharged`

Relacionado: [[Episode Discharge Process]]

Primero pruebo como todo en true y luego lo cambio a discharged.

Contacto de prueba: https://app.hubspot.com/contacts/7712148/record/0-1/140620173319

```ruby
eps = Episode
 .joins(patient: :account)
 .where(status: :active, scheduling_enabled: true)
 .where.not(account: { hubspot_id: nil })

fep = eps.second

ap fep.patient.account.hubspot_id
ap fep.active?
ap fep.most_recent_care_plan?
```

Primero actualicé el teléfono:
```ruby
fep.update(phone_number: "3013972802")
```

Verifiqué que se haya actualizado como se espera. Luego lo pasé a `discharged`:
```ruby
fep.update(
  status: :treatment_completed,
  treatment_completed_reason: :goals_met
)
```

## Specs

Estos dos fallaban:
```bash
./spec/requests/graphql/mutations/scheduling/scheduling_appointments_performance_spec.rb:66

./spec/requests/graphql/mutations/scheduling/scheduling_appointments_performance_spec.rb:127
```

Resulta que fallan porque el método `most_recent_care_plan?` ejecuta una query en la base de datos al invocar a `patient.recent_care_plan`.

En estas dos pruebas, al crear el nuevo care plan se llama a `most_recent_care_plan?` en el callback `sync_hubspot_active_status` así que habrá una query adicional.

## Sync manual mediante el botón "sync with HubSpot"

Resulta que esto es una forma de forzar los updates a HubSpot. No sabía que existía. Estas dos propiedades también tienen que actualizarse en esa parte.

La clase que gestiona esto es `Hubspot::SyncPatientService`. Para agregar estos datos hay un método llamado `add_care_plan_data`. Ahí es donde se agrega el código.

### Pruebas

Se busca un paciente como en los pasos anteriores y desde el perfil se clica el botón "sync with Hubspot" que está en la sección Actions.
