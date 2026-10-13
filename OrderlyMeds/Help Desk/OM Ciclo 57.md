# OM Ciclo 57

> [!Info]
> Del Jueves 8 al Miércoles 21 de Octubre.

## Casos de problemas de Sermorelin en MedPicker 🟢

Etiquetas: #om_bundle_issue #om_bundle_manually_rejected 

Esto es lo mismo que pasó en OM-11617 -> [[OM Ciclo 56#Detalle de OM-11617 - Not at the Pharmacy - Bundle Rejected 🟢]]

Casos:

- OM-11638
- OM-11779

Capturas:

<figure>
    <img src="./attachments/OM-11638.png" />
    <figcaption>Caso OM-11638</figcaption>
</figure>


<figure>
    <img src="./attachments/OM-11779.png" />
    <figcaption>Caso OM-11779</figcaption>
</figure>


### Solución: Tenían que corregir el MedPicker

Cuando le fui a escribir a Sarah se me dio por revisar antes y pude hacer los resubmits.

## Caso OM-11848 - Unable with Current Medication 🟢

Etiquetas: #om_unable_to_continue_with_current_medication

Este error cuando la persona va a seleccionar tratamiento:
![[OM_11848.png]]

Claudio dice que pasa porque:
- para mostrarle su tratamiento actual (Tirzepatide inyectable), la app le pide a MedPicker las opciones para ella y luego se queda solo con las que existen en nuestra tabla de medicamentos (`Salesforce::Medication`).
- MedPicker sí le recomendó 3 opciones de Tirzepatide 12.5mg de PerfectRX, pero ninguna se importó nunca a esa tabla, así que la app las descarta. Solo quedan Semaglutide y Zepbound, la app concluye que su tratamiento "no está disponible" y muestra el modal.

Claudio está diciendo de importar esos MedIds como `Salesforce::Medication` pero no sé si deba hacer eso.

### Solución: crear los medicamentos como Salesforce::Medication

Hoy, 13 de Octubre, volví a revisar y sí dejaba llegar a la parte de seleccionar medicamento. De comprarla. Así que le pedí a claudio una consulta para comprobar que la cosa estuviera en orden.

Esta es la query:
```ruby
ids = %w[qf8LOJi9avCNF02x7Wb9cNDkPDUnHiGk wGXBEaL06nqTuiwyFggTTObrcWXeBJpL ZRcLdJ595bOcmiBwa63afnq9jxHurZap]
pp Salesforce::Medication.where(med_id: ids).includes(:product).map { |m|
  {med_id: m.med_id, status: m.status, sku: m.product&.sku, desc: m.description,
   rx_strength: m.rx_strength, weeks: m.weeks_of_supply, imported_at: m.imported_from_medpicker_at}
}
```

y la salida fue:
```ruby
[
 {med_id: "qf8LOJi9avCNF02x7Wb9cNDkPDUnHiGk",
  status: "Active",
  sku: "TIRZ-1M-HIGH",
  desc: "PerfectRX Tirzepatide Injection 12.5mg (1 Month) Tirzepatide/B3/B5 20mg/5mg/10mg/1mL - 2.5mL",
  rx_strength: 12.5,
  weeks: 4.0,
  imported_at: 2026-10-13 14:27:44.000000000 UTC +00:00},
  
 {med_id: "ZRcLdJ595bOcmiBwa63afnq9jxHurZap",
  status: "Active",
  sku: "NULL_PRODUCT",
  desc: "PerfectRX Tirzepatide Injection 12.5mg (2 Months) Tirzepatide/B3/B5 20mg/5mg/10mg/1mL - 3mL",
  rx_strength: 12.5,
  weeks: 8.0,
  imported_at: 2026-10-13 14:28:54.000000000 UTC +00:00}
]
```

Hubo el problema del medicamento que quedó con SKU NULL_PRODUCT y hubo un tercer MedId que no se creó. Igual con el MedId que quedó bien se hubiera podido seguir pero decidí corregir todo.

Esto fue lo que corrí:
```ruby
mapping = {
  "qf8LOJi9avCNF02x7Wb9cNDkPDUnHiGk" => "TIRZ-1M-HIGH",
  "wGXBEaL06nqTuiwyFggTTObrcWXeBJpL" => "TIRZ-2M-HIGH",
  "ZRcLdJ595bOcmiBwa63afnq9jxHurZap" => "TIRZ-2M-TITRATE"
}
products = Salesforce::Commerce::Product.where(sku: mapping.values).index_by(&:sku)
raise "missing products: #{mapping.values - products.keys}" unless products.size == 3

missing = mapping.keys - Salesforce::Medication.where(med_id: mapping.keys).pluck(:med_id)
provider = Salesforce::ImportMedicationService::SalesforceObjectProvider.new(warn_on_ambiguous_product: false)

Salesforce::Base.transaction do
  imports = Salesforce::MedpickerImporter.fetch_medications_bulk(missing)
  pp(missing:, returned_by_medpicker: imports.map { it.medication_mapping.medpicker_med_id })
  imports.each { Salesforce::ImportMedicationService.call(it, salesforce_object_provider: provider) }

  Salesforce::Medication.where(med_id: mapping.keys).each do |med|
    med.update!(product: products.fetch(mapping.fetch(med.med_id)), status: "Active")
  end
end
```

Verifiqué con esto:
```ruby
pp Salesforce::Medication.where(med_id: mapping.keys).includes(:product).map { [it.med_id, it.status, it.product.sku] }
```

que sacó:
```ruby
[["qf8LOJi9avCNF02x7Wb9cNDkPDUnHiGk", "Active", "TIRZ-1M-HIGH"],
 ["ZRcLdJ595bOcmiBwa63afnq9jxHurZap", "Active", "TIRZ-2M-TITRATE"],
 ["wGXBEaL06nqTuiwyFggTTObrcWXeBJpL", "Active", "TIRZ-2M-HIGH"]]
```


Cuando volví a correr la query inicial de verificación dio esto:
```ruby
[
{med_id: "qf8LOJi9avCNF02x7Wb9cNDkPDUnHiGk",
  status: "Active",
  sku: "TIRZ-1M-HIGH",
  desc: "PerfectRX Tirzepatide Injection 12.5mg (1 Month) Tirzepatide/B3/B5 20mg/5mg/10mg/1mL - 2.5mL",
  rx_strength: 12.5,
  weeks: 4.0,
  imported_at: 2026-10-13 14:27:44.000000000 UTC +00:00},
  
 {med_id: "ZRcLdJ595bOcmiBwa63afnq9jxHurZap",
  status: "Active",
  sku: "TIRZ-2M-TITRATE",
  desc: "PerfectRX Tirzepatide Injection 12.5mg (2 Months) Tirzepatide/B3/B5 20mg/5mg/10mg/1mL - 3mL",
  rx_strength: 12.5,
  weeks: 8.0,
  imported_at: 2026-10-13 14:28:54.000000000 UTC +00:00},
  
 {med_id: "wGXBEaL06nqTuiwyFggTTObrcWXeBJpL",
  status: "Active",
  sku: "TIRZ-2M-HIGH",
  desc: "PerfectRX Tirzepatide Injection 12.5mg (2 Months) Tirzepatide/B3/B5 20mg/5mg/10mg/1mL - 2.5mL",
  rx_strength: 12.5,
  weeks: 8.0,
  imported_at: 2026-10-13 14:45:49.000000000 UTC +00:00}
  ]
```

Los tres MedIds como `Salesforce::Medication`.

## Caso OM-11828 - Error de Scheduled Clinical Encounter en Salesforce 🟢

Etiquetas: #om_scheduled_clinical_encounter_exists 

El error cuando intenté hacer un ResubmitToMSO:
![[OM_11828.png]]

> `Patient::Checkin::ClinicalEncounterScheduler::NotSchedulable` A scheduled clinical encounter already exists


El problema es que el Member Period tenía dos Clinical Encounters creados casi que al mismo tiempo:
![[OM_11828.b.png]]

Donde el CE con omid `"01a1122b-c571-7e67-b8f5-a652e9d40626"` era el que necesitaba cancelarse pero el sistema estaba agarrando el que aparece después. Que ya estaba cancelado.

Entonces el fix fue hacer el resubmit desde consola pasando este omid (el del CE en scheduled). Así.

Primero se confirma que se devuelve lo que se quiere. Este era un caso de cambio de medicamento entonces queríamos...
```ruby
ce = Salesforce::ClinicalEncounter.find("01a1122b-c571-7e67-b8f5-a652e9d40626")
p = Salesforce::ResubmitToMso.preview(clinical_encounter: ce, overrides: {prescription_strength: 10, weeks_of_supply: 4})

pp p.validation_errors, p.matching_product&.name, p.recommended_medication&.then { {med_id: it.med_id, rx_strength: it.rx_strength, weeks: it.weeks_of_supply} }
```

...esta salida
```ruby
[]
"1-month Tirzepatide (High Dosage)"
{med_id: "EwS6VvRRwnBZhvdE775hKexfka4kKTwm", rx_strength: 10.0, weeks: 4.0}
```

Luego se ejecuta manualmente pasando el Clinical Encounter que nos interesa:
```ruby
r = Salesforce::ResubmitToMso.execute(clinical_encounter: ce, overrides: {prescription_strength: 10, weeks_of_supply: 4})
```

Verificación posterior:
```ruby
mp = ce.member_period.reload
puts mp.status

mp.clinical_encounters.order(:start_date).last(3).each { puts [it.omid, it.status, it.source_system_identifier].join(" | ") }; nil
```

La salida es algo como esto:
```
01a1122b-c571-7e67-b8f5-a652e9d40626 | Cancelled | 37ef5b8c-2ca3-43d4-b88f-2ff0d29bceaa
01a1122c-55ed-7410-8112-4274ef9d50c7 | Cancelled |
01a135e9-f491-73c4-b557-ee9e84e68605 | Scheduled | bd9425e5-f961-44b9-b1ce-5cd3e4785d05
```

donde ya vemos al `01a1122b` ya en estado Cancelled.

## Migraciones manuales de OP a Salesforce

Etiquetas: #om_migration_op_to_sf 

> [!Info]
> Ver también [[Codigo de Soluciones#Código para solucionar casos en Orderlymeds#Migrar Contacto de Ontraport a Salesforce]]

Casos:
- OM-11741
- OM-11757

Al clicar el botón "Migrate to Salesforce" no pasaba nada al tiempo. Así que toca manual. Estos son los pasos para hacerlo desde consola.

### Diagnóstico

```ruby
a = Account.find("019adcd9-b070-73fb-8ec3-f9f1a46322b4")
cid = a.ontraport_contact.nk

[a.migrated_to_salesforce?, a.salesforce_account_nk, a.prescriber]
```

Salida de ese caso:
```ruby
[false,
 nil,
 #<Prescriber:0x00007fe5851ede68
  id: "0196abf9-1e99-7c04-b77c-48563cde35aa",
  name: "Care Validate",
  created_at: "2025-05-07 18:19:12.409560000 +0000",
  updated_at: "2025-05-27 17:20:19.649489000 +0000",
  nk: "733">]
```

Luego se valida el grupo de prescribers y la flag correspondiente:
```ruby
group_id, flag = a.prescriber.care_validate? ?
  [Salesforce::CareValidateMigrationRolloutJob::CARE_VALIDATE_GROUP_ID, :enable_care_validate_salesforce_migrations] :
  [Salesforce::BelugaHealthMigrationRolloutJob::BELUGA_HEALTH_GROUP_ID, :enable_salesforce_migrations]

[group_id, flag, Features.enabled?(flag)]
```

Salida de esto:
```ruby
[987, :enable_care_validate_salesforce_migrations, true]
```

Luego miramos si al cuenta está en ese grupo (esto suele ser la causa que no funcione el botón en Success)
```ruby
client = RateLimited::OntraportApiClient.new(limiter: Sidekiq::Limiter.unlimited)
client.get_contacts_by_group(group_id, ids: cid.to_s, perform_all: 1).any?
```

Salida:
```ruby
false
```

Finalmente, verificamos que no haya errores al preparar la clase que hace la migración:
```ruby
ci = OntraportMigration.fetch_contact(cid)
ci.valid?
```

Que debe sacar: `true`

Además este saca errores si los hubiere:
```ruby
pp ci.validation_errors
```

Salida:
```ruby
{by_instance: [], by_class: {}}
```

Este último lista los Member Periods que se crearían:
```ruby
ci.member_period_mappings.map { s = it.ontraport_script; [s.id, s.outcome, s.next_consult, s.case_id] }
```

```ruby
[[690892, "Cancelled", 2026-10-01 16:00:00 UTC, nil],
 [696836, "Shipped", 2025-12-01 17:00:00 UTC, "1d374095-7de6-4ed2-b195-802f592592f2"],
 [701881, nil, 2026-02-25 02:07:47 UTC, "1d374095-7de6-4ed2-b195-802f592592f2"],
 [851853, "Delivered", 2026-05-13 03:52:05 UTC, "1d374095-7de6-4ed2-b195-802f592592f2"],
 [942547, nil, nil, nil],
 [942548, nil, nil, nil],
 [942645, "Active", 2026-10-01 16:00:00 UTC, nil]]
```

### Migrar

Si no hubo errores, se corre este comando (usa las variables de los comandos anteriores):
```ruby
Ontraport::Meta.with_rate_limiting(limiter: Sidekiq::Limiter.unlimited) do
  Salesforce::OntraportAccountImporter.call(
    ontraport_contact_id: cid,
    contact_import: ci,
    local_account: a,
    logger: Logger.new($stdout)
  )
end
```

La salida que se espera es `:imported`.