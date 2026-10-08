# OM Ciclo 56

> [!Info]
> Del Jueves 24 de Septiembre al Miércoles 7 de Octubre.

## Caso OM-11571 - Reiniciar Script para Starter Pack 🟢

Etiquetas: #om_starter_pack_ontraport 

Esto es una continuación de [[OM Ciclo 55#Casos de Starter Pack en Ontraport 🟢ℹ️]]

Para este caso, agregué los automations descritos pero el Checkin no mostraba la pregunta sobre la última vez que tomó medicina.

Así se veía el Checkin:

> [!Warning]
> Nota como el diseño se ve diferente al checkin habitual.

![[om.incomplete.checkin.png]]

### Solución: Fabian migró el cx a Salesforce

Y ahora sí carga el Checkin que se espera. Mira lo diferente que es con respecto al de OP:
![[om.correct.checkin.png]]

## Caso OM-11599 - CV Request con 5 validaciones Flagged 🟢

> [!Note]
> Conclusión: reiniciaron el Checkin.

Etiquetas: #om_care_validate_needs_review #om_care_validate_flagged

Caso en el que el Script queda en Pharmacy Selected pero el Request tiene el estado `needs_review`. Al buscar Flagged Case Decisions se ve que tiene las 5 validaciones en error:

- **Base medication mismatch**: _CareValidate decision approval does NOT have the same base medication as the recommendation. Product not found. Error occured._
- **Contraindications mismatch**: _Contraindications do not match. Error occured._
- **Unknown MedId**: _Known med ids do not match. Error occured._
- **Weeks supply mismatch**: _Number of weeks do not match. Error occured._
- **Missing MedId (external_id)**: _CareValidate decision approval does NOT have a prescribed med_id. Error occured._

### El Problema: Desde Care Validate no eligieron medicina del MedPicker

Claudio ayudó a investigar y encontró que el webhook `ADD_CASE_DECISION` no tiene lo necesario en la llave `medInfo`. Así está:
```json
"medInfo": [
	{
		"id": "f07a4e00-529d-4732-a159-7ac86c14c5fc",
		"medicine": "Tirzepatide/B12 10mg/1mg/0.5mL - 2mL",
		"dosage": "Month 1: Inject 0.25ml (25 units) subcutaneously once weekly (0.25ml is 2.5mg per week). Month 2: Inject 0.5ml (50 units) subcutaneously once weekly (0.5ml is 5mg per week).",
		"refillCount": 0,
		"pharmacyInstructions": "",
		"dosingFrequency": "Weekly",
		"externalId": null,
		"isRefill": true,
		"treatmentPeriod": "Week 8",
		"isPriorAuthRequested": false
	}
]
```

Donde el problema está con el campo `externalId`. Ese es el MedId.

Así se ve en un caso que sí tiene este payload completo (caso es OM-11449):
```json
"medInfo": [
	{
		"id": "dfc161ac-dd45-493c-8850-83bde6e80c40",
		"medicine": "Tirzepatide/B12 10mg/0.5mg/1mL - 2mL",
		"dosage": "Inject 0.5ml (50 units) subcutaneously once weekly (0.5ml is 5mg per week).",
		"refillCount": 0,
		"pharmacyInstructions": "",
		"dosingFrequency": "Weekly",
		"externalId": "fESwPH3bs2HN6VMa4pF6P3e2DzLfCsxw",
		"isRefill": true,
		"treatmentPeriod": "Week 4",
		"isPriorAuthRequested": false
	}
]
```

### Solución: Pedir a CareValidate que arregle y mande el webhook de nuevo

Mandé mensaje en el hilo del caso. Esperando a ver qué dicen.

## Caso OM-11594 - Orden faltante luego de Import desde Ontraport a Salesforce 🟢

Etiquetas: #om_missing_orders 

> [!Important]
> Esto pasó porque están migrando cx de Ontraport a Salesforce cuando estos tienen aún un Script en curso.
> 
> Cuando empecé a revisar el Member Period del Script no tenía el Medication Request así que me tocó reimportar y actualizar el estado del MedRequest y del Member Period.
>
> Al final, esto también afecto a que se creara el OrderSummary en Salesforce. Tocó reimportar para poder mostrar la orden en el portal del paciente.

Mismo caso de ordenes faltantes. Al Member Period le faltaba el Medication Request. Luego de reimportar seguía sin mostrarse la orden. Faltaba un OrderSummary por crear.

### Solución: reimportar OrderSummary

Con este código sacado y ajustado desde `Salesforce::OntraportAccountImporter`.

#### Verificaciones

Verifiqué con esto:
```ruby
acct = Account.find("019fc35f-b8f2-7033-a928-46480fcd2915").salesforce_account

acct.order_summaries.includes(:member_period).map do |os|
  mp = os.member_period
  o  = Patient::Connectors::Salesforce::Order.new(order_summary: os)
  
  {
    os: os.order_number,
    total: os.try(:grandtotalamount),
    mp: mp&.omid,
    mp_status: mp&.status,
    op_outcome: mp&.ontraport_imported_outcome,
    outcome: o.outcome,
    valid: o.is_a_valid_order?
  }
end

acct.member_periods.map { {omid: it.omid, status: it.status, op_outcome: it.ontraport_imported_outcome, os_count: it.order_summaries.size} }
```

La salida de eso solo mostró un OrderSummary cuando la expectativa eran dos.

También se verificó que no hubiera una orden suelta para esa cuenta con:
```ruby
acct = Account.find("019fc35f-b8f2-7033-a928-46480fcd2915").salesforce_account

Salesforce::Order.where(account__omid__c: acct.omid).map { {omid: it.omid, mp: it.memberperiodid__r__omid__c, imported: it.imported_from_ontraport_at, summaries: Salesforce::OrderSummary.where(originalorder__omid__c: it.omid).count} }
```

#### Código Reimportar

Se hacen verificaciones previas:
```ruby
acct    = Account.find("019fc35f-b8f2-7033-a928-46480fcd2915").salesforce_account
mp      = acct.member_periods.find_by!(omid: "01a09d0c-c1ce-71d4-8ec0-529d33d716ce")
ci      = OntraportMigration.fetch_contact(688348)
mapping = ci.order_mappings.find { it.ontraport_script_id.to_i == 941434 }

medication      = Salesforce::Medication.find(mapping.med_id)
product         = medication.product
pricebook       = Salesforce::Commerce::Pricebook.standard
pricebook_entry = product.pricebook_entries.find_by!(pricebook:)

{mp_script: mp.ontraport_script_id, existing_orders: Salesforce::Order.where(account: acct, member_period: mp).count,
 med_id: mapping.med_id, mr_med: mp.medication_requests.map(&:medicationid), product: product.name,
 order_attrs: mapping.mapped_order_attributes, item_attrs: mapping.mapped_order_item_attributes}
```

Crear el OrderSummary:
```ruby
order = Salesforce::Base.transaction do
  Salesforce::Order.create!(
    account: acct, member_period: mp, pricebook:,
    status: Salesforce::Order::STATUS_DRAFT,
    imported_from_ontraport_at: Time.current,
    **mapping.mapped_order_attributes
  ).tap do |order|
    order.update!(effective_date: order.effective_date || Time.current) unless order.effective_date

    delivery_group = Salesforce::OrderDeliveryGroup.create!(
      order:,
      order_delivery_method: Salesforce::OrderDeliveryMethod.standard,
      deliver_to_name: [acct.first_name, acct.last_name].join(" "),
      deliver_to_first_name: acct.first_name,
      deliver_to_last_name: acct.last_name,
      email_address: acct.person_email,
      phone_number: acct.person_mobile_phone,
      deliver_to_street: acct.person_mailing_street,
      deliver_to_city: acct.person_mailing_city,
      deliver_to_state_code: acct.person_mailing_state_code,
      deliver_to_postal_code: acct.person_mailing_postal_code
    )

    Salesforce::OrderItem.create!(
      order:, order_delivery_group: delivery_group, product:, pricebook_entry:,
      description: product.name, **mapping.mapped_order_item_attributes
    )

    order.update!(status: Salesforce::Order::STATUS_ACTIVATED)
  end
end
```

Tarda varios minutos mientras se replica en Salesforce. Cuando esta ultima verificación saque los order summaries que corresponden, estaremos cerca de tenerlo solucionado:
```ruby
acct.order_summaries.includes(:member_period).map { o = Patient::Connectors::Salesforce::Order.new(order_summary: it); {os: it.order_number, mp: it.member_period&.omid, outcome: o.outcome, valid: o.is_a_valid_order?} }
```

## Caso OM-11531 - Migrar a Salesforce después de rollback a Ontraport 🟢

Etiquetas: #om_migration_op_to_sf 

Esto fue una cuenta que se le hizo el "rollback" manual a Salesforce. Para solucionar estos casos se tomó la decisión de otorgar créditos en Salesforce y crear un nuevo Member Period. Así que necesitaba llevar la cuenta de nuevo a Salesforce.

Había clicado el botón "Migrate to Salesforce" del perfil en Success pero no pasaba nada. Así que la otra alternativa fue llenar manualmente el valor de `salesforce_account_nk`.

> [!Note]
> Cómo migrar a Salesforce
> 
> Estas son las opciones que me dio Fabian:
> - Clicar el botón en el perfil en Success y esperar
> - Buscar la cuenta en Salesforce, copiar el `omid` y actualizar `salesforce_account_nk`
> - Hacer un recorrido manual de `import_account_from_ontraport` hasta encontrar un posible error

Le actualicé el valor con el `omid` y ya quedó enlazada.

## Casos de resubmit en Salesforce que dan "Unable to find matching product" 🟢ℹ️

Etiquetas: #om_unable_to_find_matching_product 

Similar como en [[OM Ciclo 50#Caso OM-9848 - Unable to find matching product 🔵ℹ️]]

Los casos:

- OM-11543: stuck in ReadyToCreateVisit
- OM-11617: not at the pharmacy

> [!Note]
> Jaime me dijo que revisara si el MedId estaba disponible en el estado del CX.

Esto dijo Slackbot sobre el problema:
> comes from Salesforce's MSO resubmission logic. ==It's trying to match a treatment order (form, weeks of supply, titrate-up, starter pack, BMI category) against the pharmacy's available product catalog, and none of the listed available products satisfy the requested combination== (e.g. requesting 12 weeks of supply with titrate up: false when only titrate-up or shorter-supply variants exist).


Y dice que se puede resolver:
> Pick one of the products actually listed as "Available" in the error message (closest match to what the customer needs)

Slackbot me ayudó a entender el error porque veía mucho texto. Ahora sé cuáles son las opciones para el resubmit al ver el texto de "Available":
![[unable.to.find.matching.product.png]]

### Detalle de OM-11543 - Stuck in Ready to Create Visit 🟢

> [!Info]
> CX de CareValidate.

Al buscar el MedId en la nueva página en el MedPicker no hay resultado. No hay 8 semanas con Titrate.

Lo importante en este caso es que estaba *stuck en ReadyToCreateVisit*. Lo que recomendó el MedPicker no importa. Ya el prescriber verá que le ofrece al cx.

#### ¿Cómo se procede?

Pregunté a Fabian cómo se hacía resubmit en este caso dijo que:
> Trata de no darle resubmit, porque eso vuelve a iniciar el proceso desde un estado anterior al error que estás viendo en imagen que pones, y probablemente haya un bug en ese flujo.
> En su lugar, intenta buscar el job que ejecuta el `visit_created`, que ocurre posteriormente a ese estado, y revisa si de esa forma funciona. Anteriormente me funcionó hacerlo de esa manera.

Y eso hice con ayuda de Claudio.

Comprobaciones:
```ruby
mp = Salesforce::MemberPeriod.find_by!(omid: "01a03111-6109-7bc0-bd43-cbb063899692")
ce = mp.clinical_encounters.last
rec = ce.med_picker_recommendation
sp = rec&.selected_products&.first
med = sp&.recommended_medication

puts "MP: #{mp.status} | CE: #{ce.omid} #{ce.status} visit_type=#{ce.visit_type} ssi=#{ce.source_system_identifier.inspect}"
puts "REC: #{rec&.omid} #{rec&.status}"
puts "MED: #{med&.med_id} | #{med&.name} | #{med&.rx_strength} -> #{med&.titration_final_rx_strength} | #{med&.status} | qty=#{sp&.recommended_quantity}"
puts "CV request: #{CareValidate::Request.where(clinical_encounter_id: ce.omid).map { [it.id, it.state, it.nk, it.case_nk] }.inspect}"
```

Se espera que:
- **MP** en `ReadyToCreateVisit` y **CE** en `Pending`, con `source_system_identifier` vacío, porque todavía no se envió.
- **REC** con `selected_products` y **MED** = `VHULlg…`, en `Active` y con `recommended_quantity`.
- **CV request**: nada, o una en un estado previo a `waiting_for_prescription`.

> [!Note]
> Se recomienda que el CV Request esté en `needs_requested_medpicker_data`.

Si todo está en orden, se ejecuta el job directamente para mover al Member Period:
```ruby
CareValidate::Scheduler::CreateVisitJob.perform_async(ce.omid)
```

#### Conclusión

El Member Period hizo todo el recorrido y ya la orden llegó a la farmacia.

### Detalle de OM-11617 - Not at the Pharmacy - Bundle Rejected 🟢

Etiquetas: #om_bundle_issue #om_bundle_manually_rejected 

> [!Info]
> CX de Beluga.

No hay resultado al revisar la variante de 20MG para el MedId seleccionado. El Member Period está en *VisitCompleted*.

#### Fallo en la validación del Bundle

```ruby
b = RxWrittenBundle.find("01a0b6eb-4726-788b-9f6b-b19c8ba0b4ff")
b.validation_runs.each do |run|
  puts "RUN #{run.test_suite}: #{run.state}"
  run.check_records.reject(&:passed?).each { puts "  FAIL #{it.name.demodulize}: #{it.message}\n    #{it.support_context.to_json}" }
end; nil
```

Salida:
```
RUN all_prescriptions_in_bundle: passed
RUN glp1: passed
RUN nadplus: passed
RUN sermorelin: failed
  FAIL QuantityMatch: RxWrittenBundle prescribed quantities do NOT match MedPicker remote quantities
```

La clave está en el mensaje:
> QuantityMatch: RxWrittenBundle prescribed quantities do NOT match MedPicker remote quantities

Así se ve el error en la página del Bundle:
![[OM_11617.png]]

#### ¿Cómo se le hace ResubmitToMSO si no se puede en Salesforce?

> [!Info]
> Esto solo es para casos donde no falle el Automation. Si falla, es por otra cosa y esto tampoco hará nad.

Hacer el resubmit to MSO mediante consola:
```ruby
Salesforce::ResubmitToMso.call(clinical_encounter: ce)
```

#### Solución: MedId no estaba activo en MedPicker 🔑

Esto es lo que parece. Ya lo está y pude continuar con el ResubmitToMSO en Salesforce.

## Caso OM-11606 - Stuck in PrescriptionWritten de PerfectRx 🟢

Etiquetas: #om_stuck_in_prescription_written #om_perfect_rx 

Lo mismo del caso [[OM Ciclo 55#Caso OM-11464 - Stuck en PrescriptionWritten y PerfectRx 🟢ℹ️]]

```ruby
client = PerfectRx::Client.new(config: PerfectRx.config, logger: Rails.logger)
p = PerfectRx::Patient.find("0194b953-d726-7e16-afcf-992ba9ffca22")
```

Luego la comparación:
```ruby
a = PerfectRx::Api::Patient.fetch(client:, external_patient_nk: p.external_nk)
b = PerfectRx::Api::Patient.fetch(client:, smart_scripts_patient_nk: p.smart_scripts_nk)
```

Salidas:
```ruby
#<PerfectRx::Api::Error:0x00007f964a558da8
 @data=nil,
 @message="Could not find patient with that ID in system.",
 @raw={"status" => "FAILURE", "message" => "Could not find patient with that ID in system.", "data" => nil},
 @status="FAILURE">
 
 #<PerfectRx::Api::Patient:0x00007f9646848710
 @conditions=[],
 @contact=
  {name: "Client Name",
   phone_number: "xxxxxxxx",
   email_address: {email: "correo@me.com", tags: ["DEFAULT"]},
   address: {line1: "address", line2: nil, city: "Cypress", state: "TX", zip: "77433"}},
 @date_of_birth=Wed, 16 Nov 1983,
 @external_id="0192fece-fb3f-7cd4-b050-d262a850a3f2",
 @first_name="Client",
 @gender=:female,
 @last_name="Name",
 @medications=[],
 @primary_insurance={},
 @secondary_insurance={},
 @smart_scripts_patient_id="1579d6e4-4299-4990-a5a9-1f000162dc43">
```

Solución:

> [!Note]
> El ID del prescription se toma de la pestaña de PerfectRx del Case Overview. El más reciente.

```ruby
p.update!(external_nk: b.external_id)

PerfectRx::ProcessPrescriptionJob.perform_async("01a0e94e-f01c-7a66-b1ec-f359cf2cfb83")
```

## Caso OM-11637 - Fallo de Beluga por Ondansetron 🟢

Etiquetas: #om_prescription_requires_review #om_ondansetron

Prescripción que no está en la farmacia porque falla porque no hay MedId para la pastilla Ondansetron.

Pregunté a CS Leads.

### Actualizaciones

- 10/1: Rhystie preguntó si se podía mandar sin la Ondansetron.
- 10/2: Devin cerró. Rhystie dijo que tienen que hacer resubmit sin la pastilla.


## Caso OM-11593 - Script reiniciar to Starter Pack - Parte 2 🟢

Etiquetas: #om_starter_pack_ontraport 

Caso de Script que necesitaba ser reiniciado para que el CX pueda responder adecuadamente para habilitar el Starter Pack. **CX en Ontraport**.

Estaba en Salesforce así que hice lo pertinente para devolverlo a Ontraport. Le dieron los créditos pero no le hice reset al Script porque iba a generar el form que está mal. El cual detallo en [[OM Ciclo 56#Caso OM-11571 - Reiniciar Script para Starter Pack 🟢]]

La solución en el caso OM-11571 fue devolverlo a Salesforce, migrar los créditos a Salesforce y crear un nuevo Member Period.

Toca hacer esto mismo porque:

- Creé un nuevo Script mediante Swagger y no le cargó el botón en el portal
- En el script anterior, agregué el Automation de regenerar la URL pero no salía el botón
- Con Claudio me doy cuenta que el problema es que el Script nuevo tiene "Next Consult" en Octubre 15 y por eso no se mostraba el botón
- Cambié la fecha a 10/1, salió el botón pero manda es al Checkin de Ontraport el cual está incompleto

Por eso creo que debo devolver a Salesforce. Y eso haré.

### Migrar cx a Salesforce

Cliqué el botón y esta vez sí se completó sin yo intervenir. También se le generó el MP en Salesforce y cuando suplanté pude ver el Checkin correcto.

Le dije a Jifrel que:
> Please have cx only complete Check In form and stop once they arrive to the Check Out page. When they're in Check Out page is when Starter Pack can be activated.

Ahora espero para activarle el Starter Pack.

## Caso OM-11666 - Missing Orders from Portal 🟢

Etiquetas: #om_missing_orders 

Le faltaban varias ordenes aunque el CX solo reportó una. El fallo fue que encontré varios Member Periods del CX sin el valor esperado en el campo `ontraport_imported_outcome`.

La solución fue ponerle el valor `Delivered`. Con eso salieron.

## Caso OM-11710 - MedChat error - No se ven los chats 🟢

Etiquetas: #om_medchat 

Cuando clica en el enlace "MedChat" en el portal se redirige al dashboard.

> [!Note]
> Resumen por Claudio:
> 
> **Causa:** la cuenta se migró de Ontraport a Salesforce el 17/09 a las 20:13 UTC, cuando el script 953182 todavía estaba en `Pharmacy Selected` y sin `date_of_visit`. Por eso el importer no creó el `ClinicalEncounter`. La visita de CareValidate ocurrió después (18/09 03:58) y quedó registrada solo en Ontraport.
>
> A las 04:04 llegó el mensaje de la Dra. Carr (webhook `ADD_CASE_COMMENT`). Como no había encounter, el handler tomó la ruta de Ontraport, `FindOrCreateChat` devolvió `nil` para la cuenta migrada y `chat.messages.create!` falló. El webhook quedó en `failed` y el mensaje nunca se guardó. Al no tener chat, `/medchat` redirige al dashboard sin mostrar error.
>
> **Arreglo:** crear a mano el `ClinicalEncounter` (CareValidate, `e151861f-2a0d-44b3-815b-84afd0029576`) en el MemberPeriod importado `a0nPm00000uio0rIAA` y reprocesar el webhook `01a0b2b0-1146-7c5d-9b9b-62b8a17eac8e`.
>
> **Riesgo sistémico:** puede afectar a cualquier paciente que se haya migrado en medio de una visita. Hay 54 webhooks `ADD_CASE_COMMENT` en `failed` y 43 en `processing` desde el 18/09, pero no sé cuántos tienen esta misma causa.


El código de solución:
```ruby
a  = Account.find("01a0a0b5-bea6-731d-84b8-1bb1ae0802bd")
sfa = a.salesforce_account
mp = Salesforce::MemberPeriod.find_by!(sfid: "a0nPm00000uio0rIAA")

ce = Salesforce::ClinicalEncounter.where(patient: sfa, member_period: mp, ontraport_script_id: 953182).first_or_initialize
ce.assign_attributes(
  start_date: Time.utc(2026, 9, 18, 3, 58, 24),
  status: "Finished",
  category: "Home Health",
  source_system: "CareValidate",
  source_system_identifier: "e151861f-2a0d-44b3-815b-84afd0029576",
  visit_type: "GLP1",
  imported_from_ontraport_at: Time.current,
  ontraport_importer_version: Salesforce::OntraportAccountImporter::IMPORTER_VERSION
)
puts({new: ce.new_record?, valid: ce.valid?, errors: ce.errors.full_messages}.inspect)
```

Si la salida es como esta:
```ruby
{new: true, valid: true, errors: []}
```

Guarda y reprocesa el webhook:
```ruby
ce.save!
puts Salesforce::ClinicalEncounter.latest_for_account(account: sfa)&.source_system_identifier
```

Reprocesar el webhook:
```ruby
w = IncomingWebhook.find("01a0b2b0-1146-7c5d-9b9b-62b8a17eac8e")
w.update!(state: "pending")
ProcessIncomingWebhookJob.new.perform(w.id)
puts w.reload.state  # => "delivered"
```

Verificación final:
```ruby
a.chats.reload.map { [it.id, it.master_id, it.messages.count] }

[["01a0fe6d-694e-7dd2-a860-d2d53382bc24", "e151861f-2a0d-44b3-815b-84afd0029576", 1]]
```


## Caso OM-11730 - Resubmit de Bundle en Ontraport - Bundle Rejected 🟢

Etiquetas: #om_beluga_not_at_pharmacy #om_bundle_manually_rejected

Caso de prescripción que no llega a la farmacia. Cuenta en Ontraport pero que fue migrada a Salesforce aún estando en curso.

El problema fue que el bundle quedó en `manually_rejected` por esto:
![[om_11730.png]]

El MedId estaba desactivado. Una vez reactivado se puede hacer el resubmit. Aquí el tema es ¿cómo le hacía resubmit a en este caso?

Claudio a la ayuda.

### Resubmit de Bundle Rejected

#### Se verificó que el MedId estuviera activo

```ruby
med = "K7S0MA7LfhsjT3Skfkw0Qqydr7ZFO9RY"
pp Medpicker.get_fulfillment_data(med_id: med, provider_id: Prescriber::BELUGA_HEALTH_PROVIDER_ID).provider_script_products_is_active
```

Para este caso devolvió `true`.

#### Encontrar el Bundle

Claudio hizo esta query:
```ruby
b = RxWrittenBundle.where(state: "manually_rejected", created_at: Time.zone.parse("2026-09-09")..Time.zone.parse("2026-09-14"))
  .find { |x| x.incoming_webhooks.any? { it.data.to_s.include?(med) } }
```

Pero igual lo podría encontrar usando el ID que está en Success:
```ruby
b = RxWrittenBundle.find("01a08be7-28dd-7187-b966-da9eabebd0e9")
```

#### Resubmit

> [!Warning]
> Debe hacerse siempre que se cumpla lo siguiente:
> - MedId activo
> - No haya otro bundle activo para el mismo `master_id`
> - Que no exista ya una orden
>
> Si pasa alguna, pues preguntarle a Claudio si se cancelan o que.

Se hace así:
```ruby
b.update_columns(state: "held")
CloseRxWrittenBundleJob.new.perform(b.id, hold_rx_written_bundles: false)
b.reload.state # esperado: "automatically_approved"
```

Comprobación:
```ruby
pp b.latest_validation_run_by_suite.transform_values(&:state)

{"glp1" => "passed", "all_prescriptions_in_bundle" => "passed"}
```

Después de esto el Script pasó a "Order at Pharmacy" y se creó una orden.

## Caso OM-11774 - Oops Error después de Purchase 🟡

Etiquetas: #om_oops_error 

> [!Info]
> Caso particular de Oops Error. Me salía cuando iba a "Select Treatment". Lo arreglé siguiendo los pasos normales. Enlazar usuario de WorkOS y crear Salesforce::CustomerUser. Sin embargo, reportaron que cuando clicaban el botón "Purchase" seguía saliendo el mismo mensaje.
>
> Viendo con Claudio la causa es el MedId de un MedRequest Importado. Estaba en modo uuid separado con guiones. Sin embargo, Claudio dice que los MedIds no siguen ese patrón.

Este es el resumen que me dio Claudio.

### Resumen del caso por Claudio 🤖

**Síntomas**
- Primero no podía hacer el check-in. Lo arreglaste el 6/oct con lo estándar: asociar la cuenta de WorkOS, crear el Salesforce customer user y corregir MPs importados de OP.
- El 7/oct CX reportó un error nuevo: la página "Oops" al dar Purchase en /checkin/purchases. El total era $0 porque un crédito CS de $298 cubre los $269.10.

**Causa**
- `create_medpicker_recommendation` falla con `ActiveRecord::ValueTooLong` (varchar(32)), así que el Purchase hace rollback y no se crea nada.
- El último MR de GLP-1 de la paciente apunta a una medicación cuyo `medid` es un UUID de 36 caracteres (`05787c05-`…).
- En producción, `medication.medid__c` acepta 50, pero las columnas `*__r__medid__c` de `medpickerrecommendation__c` siguen en 32. Es un desfase del mapeo de Heroku Connect.
- `db/heroku_connect_schema.rb` dice 50 y no refleja producción.

**Descartado**: créditos (cuadran en $298), estado del MP (el activo está en `ReadyForProductSelection`) y datos de entrega (completos).

#### Comando de debug (dry run del Purchase, siempre hace rollback)

Reproduce exactamente lo que hace el botón Purchase y muestra el paso, el SQL y los valores que fallan. Sirve para cualquier paciente: solo cambia el ID de la cuenta.

```ruby
sa = Account.find("019d1b7c-6d74-7d4b-be6f-e714de89e5e2").salesforce_account
form = Patient::Checkin::PurchaseForm.new(salesforce_account: sa)
form.populate_previous_selections
form.treatment ||= form.treatment_picker.default_treatment
puts "valid=#{form.valid?} #{form.errors.full_messages.inspect} product=#{form.product} total=#{form.total}"

step = nil
Salesforce::Base.transaction do
  begin
    step = "medpicker_recommendation"; form.create_medpicker_recommendation
    step = "create_order";             r = form.create_order
    step = "state_transition";         r.needs_payment? ? form.member_period.mark_ready_for_order_payment! : form.member_period.mark_order_payment_submitted!
    puts "OK needs_payment=#{r.needs_payment?}"
  rescue => e
    err = e.is_a?(ActiveRecord::StatementInvalid) ? e : (e.cause if e.cause.is_a?(ActiveRecord::StatementInvalid)) || e
    puts "STEP=#{step}"
    puts "#{err.class}: #{err.message}"
    puts "SQL: #{err.try(:sql)}"
    puts "BINDS: #{err.try(:binds)&.map { |b| b.respond_to?(:name) ? "#{b.name}=#{b.value_before_type_cast.to_s.truncate(80)} (#{b.value_before_type_cast.to_s.length})" : b.inspect }&.inspect}"
    puts e.backtrace.grep(%r{/app/(app|lib)/}).first(10)
  ensure
    raise ActiveRecord::Rollback
  end
end
```

Esta fue la salida del dry-run:
```ruby
valid=true [] product=0198c32e-8319-7eba-856a-10621bd78d27 total=0.0
OK needs_payment=false
```

#### Comando de corrección (cambiar la medicación del MR)

Mismo nombre y producto, con medid de 32. Corre antes el dry run de arriba con este update! dentro de la transacción para confirmar que sale OK, y pide aprobación porque toca un MR Completed.

```ruby
mr = Salesforce::MedicationRequest.find_by(omid__c: "01a0b4ca-b9b6-767d-909b-61e465766b89")
mr.update!(medication__omid__c: "019b03fb-f9c6-7461-9dd2-a3609030031f")  # antes: 019b03fb-f457-7871-aac9-14f68f90892f
mr.reload.medication.medid__c  # => "5NuaScOPVmaE68Gi5D9b0RdPYspSnM8g"

# Revert:
# mr.update!(medication__omid__c: "019b03fb-f457-7871-aac9-14f68f90892f")

Limpieza de MemberPeriods viejos (después de que compre)
Cancela los 3 MPs viejos en ReadyForProductSelection. El activo es 01a0e92f… y no se toca.

%w[01a0cb22-7333-7111-a376-a9179e6df114 01a0d9af-cb61-7ea6-a6b0-07215967b93d 01a0da5a-b2a5-787e-b459-d50ca8cecbf8].each do |id|
  mp = Salesforce::MemberPeriod.find_by(omid__c: id)
  next puts("#{id} tiene #{mp.orders.count} órdenes, revisar") if mp.orders.exists?
  puts "#{id} #{mp.status} -> #{mp.cancel ? 'Canceled' : "FAILED #{mp.errors.full_messages}"}"
end
```