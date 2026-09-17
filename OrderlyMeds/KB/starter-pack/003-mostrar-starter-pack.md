# ¿Cuándo se muestra el 2-Month option en el Check out?

## 1. El dato base: `Product.is_starter_pack`

`app/models/salesforce/commerce/product.rb:67` — alias de `IsStarterPack__c` en Salesforce, obligatorio (`validates :is_starter_pack, inclusion: {in: [true, false]}`, línea 78). Es una bandera del producto, no un objeto aparte.

## 2. Productos starter vigentes (`pricing/current.json`)

6 SKUs, 4 activos:

| SKU | Nombre | Semanas | Precio std | Activo |
|---|---|---|---|---|
| `SEMA-2M-STARTER` | Semaglutide 2-month Starter Pack | 8 | $149 | ✅ |
| `TIRZ-2M-STARTER` | Tirzepatide 2-month Starter Pack | 8 | $299 | ✅ |
| `SEMA-3M-STARTER` | Semaglutide 3-month Starter Pack (ExtendedBud) | 12 | $224 | ✅ |
| `TIRZ-3M-STARTER` | Tirzepatide 3-month Starter Pack | 12 | $449 | ✅ |
| `NADPLUS-3M-STARTER` | 3-month NAD+ Starter Pack | 12 | $525 | ❌ |
| `SERM-3M-STARTER` | 3-month Sermorelin Starter Pack | 12 | $450 | ❌ |

## 3. Quién los puede comprar. Se controla por pricebook, no por código

Los 3M starter **solo** están activos en los pricebooks `NewNonTransfer` y `Restarting` (B2B, B2C y Employee). Los clientes `Existing` y `NewTransferIn` solo ven los 2M starter. Eso viene del cambio `7ddd8049c` "Only offer starter packs to new and restarting customers" (2026-03-10, solo datos de pricing) y luego `4dbdc6400` "New 2M Starter Plans" (2026-06-11), que reactivó los 2M para todos.

## 4. Camino del paciente: `IsStarterPlanOnly__c`

- Pregunta del check-in: *"When did you last take your prescribed weight loss medication?"* → si respondió "más de dos meses", `PatientCheckin__c.IsStarterPlanOnly__c = true` (`questionnaires/check-in.yml:344`).
- `app/services/patient/checkin.rb:47` — si es starter-plan-only, el MemberPeriod pasa a `customer_lifecycle_stage: "Restarting"` (que a su vez abre el pricebook con los 3M starter).
- `app/services/patient/checkin/med_picker_recommendation_builder.rb:41` — en ese caso **no** se envían los campos de medicación previa (strength, form, treatment, dosage_feedback) a MedPicker, para que arranque desde la dosis más baja.
- La vista lo explica al paciente: `app/views/patient/checkin/purchases/new.html.erb:20` ("Start over at the lowest dose…").
- En el selector de plan se etiqueta con el sufijo `" (starter)"`: `app/forms/patient/checkin/purchase_form/product_picker.rb:45`.

## 5. Resubmit to MSO — override "Starter Dose"

`app/services/salesforce/resubmit_to_mso.rb`:
- Opción de staff `is_first_month`, etiquetada "Starter Dose" (línea 133).
- Si está activa y no hay med_id seleccionado, se **limpian** todos los campos de medicación para forzar el `StartingPersonalizedHandler` (líneas 351-360).
- El matching de producto cambia: solo por `treatment_form` + `is_starter_pack: true`, porque los starter packs difieren en `weeks_of_supply`, `titrate_up` y `bmi_category` (líneas 520-525).

## 6. Notas sueltas

- `a8084fbd7` arregló un bug donde la importación de medicaciones asignaba producto nulo cuando los SKUs `STARTER`/`TITRATE` resultaban ambiguos.
- `266d8f285` (2026-08-18) eliminó el campo `isStarterDose` de la importación de MedPicker por no usarse.
- Hay restos legacy de Ontraport con otra nomenclatura ("Just Getting Started (starter doses)", `StarterStatus__c`) en `lib/salesforce/webhooks/person_account_to_zapier_transformer.rb:86`.
- El FAQ público (`config/locales/internal/api/v1/supports/faq/en.yml:96`) cita los precios de los programas 2-month starter: $299 tirzepatide / $149 semaglutide.
