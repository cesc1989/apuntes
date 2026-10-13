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