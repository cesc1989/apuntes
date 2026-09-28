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

## Caso OM-11599 - CV Request con 5 validaciones Flagged 🟡

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

## Casos de resubmit en Salesforce que dan "Unable to find matching product" 🟡ℹ️

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

### Detalle de OM-11543 - Stuck in Ready to Create Visit 🟡

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

### Detalle de OM-11617 - Not at the Pharmacy 🟡

> [!Info]
> CX de Beluga.

No hay resultado al revisar la variante de 20MG para el MedId seleccionado. El Member Period está en *VisitCompleted*.

#### Validación de fallo del Bundle

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

#### ¿Cómo se procede?

Hacer el resubmit to MSO mediante consola:
```ruby
Salesforce::ResubmitToMso.call(clinical_encounter: ce)
```