# OM Ciclo 55

> [!Info]
> Del Jueves 10 al Miércoles 23 de Septiembre.

## Caso OM-11347 - MP Stuck at ReadyToCreateVisit de CareValidate 🟢

Etiquetas: #om_stuck_in_ready_to_create_visit

Casos con el mismo problema (no necesariamente misma solución):

- OM-9396 -> [[OM Ciclo 48#Caso OM-9361 - MP Stuck in ReadyToCreateVisit 🟢]]
- OM-7070

Iba a hacer lo del caso relacionado, sin embargo, el MP de este caso es de Care Validate. Lo que hice fue un resubmit normal.

### Actualizaciones

- Después del Resubmit to MSO cambió el estado a "VisitCreated"

## Caso OM-11346 - Stuck in Submitted que pasa Script Error 🟢

Etiquetas: #om_stuck_in_submitted #om_script_error #om_no_matching_recommendations #om_needs_requested_medpicker_data 

Típico caso de Stuck in Submitted, sin embargo, después de hacer el resubmit pasó a Script error:
> No matching recommendations for these patient preferences.

Cuando reviso el CareValidate::Request en la parte de MedPicker data no hay nada.

Caso similar es [[OM Ciclo 50#Caso OM-9790 - Script error con needs_requested_medpicker_data 🟢ℹ️]]

### Solución: copiar medid de CV Request en cancelada

Hice lo mismo que en el caso OM-9790. Fui al request cancelado y copié el MedId y luego corrí "Fix Medpicker Selection". Luego revisé y el Script cambió a "Pharmacy Selected".

## Caso OM-11414 - Stuck in ReadyToCreateVisit de Beluga 🟢

Etiquetas: #om_stuck_in_ready_to_create_visit

Al contrario que OM-11347 al inicio de este doc, este como sí es de Beluga intenté el troubleshoot pero dio el mismo problema del status 400.

Así que mandé el mensaje tal cual hice en OM-9396 -> [[OM Ciclo 48#Caso OM-9361 - MP Stuck in ReadyToCreateVisit 🟢]]

### Paso 1: intentar crear la visita

Esto es lo que sugiere el Notion:
```ruby
mp = Salesforce::MemberPeriod.find_by(omid: "OMID")
ce = mp.clinical_encounters.last
BelugaHealth::Scheduler::CreateVisitJob.new.perform(ce.id)
```

pero devolvió el mensaje:
> BelugaHealth#visit_form_submission failed with status 400: Patient not eligible for this visitType

El mensaje enviado a CS:
> When trying to create the visit, Beluga returns the error: _"Status 400: Patient not eligible for this visit."_ Please contact the provider to determine why the patient is not eligible for the visit.


## Caso OM-11407 - Starter Pack 🟢

Etiquetas: #om_starter_pack_checkin 

Un CX que necesita el Starter Pack. El MP estaba en "Ready For Product Selection" y el Checkin completo. Sin embargo, cuando lo activé con:
```ruby
mp = Salesforce::MemberPeriod.find_by(name: "MP-00664447")
mp.update!(customer_lifecycle_stage: "Restarting")

check_in = mp.patient_checkins.last
check_in.update!(is_starter_plan_only: true)
```

E iba a seleccionar el producto salía un mensaje de que el CX no era elegible. Luego volví a revisar y se hizo otro Checkin que no tiene el starter pack activado pero sale la opción de 2 meses...

![[OM_11407.png]]


## Caso OM-11427 - Reroute to Beluga 🟢ℹ️

Etiquetas: #om_reroute_to_beluga 

Necesito cambiar el prescriber de un CX a Beluga y hacer el resubmit.

Hay que correr estos comandos:
```ruby
script = ::Ontraport::Meta::Script.get_by_id(950674)
request = CareValidate::Request.find("01a082b8-e322-799a-bea5-8bc8d40a49c2")
icwhs = IncomingWebhook.where(id: request.incoming_webhook_ids)
wh = icwhs.first
wh.update!(state: "pending")
request.update!(state: "needs_crm_update")

request.send_to_beluga!
CareValidate.retry_via_beluga(contact: ::Ontraport::Meta::Contact.get_by_id(script.contact))
ProcessIncomingWebhookJob.new.perform(wh.id)
```
que hace lo siguiente:
- Ubicar el script para poder cambiar el prescriber del Contacto en Ontraport
- Ubicar la Request de Care Validate
	- Y el webhook que sea de tipo "ontraport"
- Cambiar el estado del webhook a "pending"

Luego terminamos con estos:
```ruby
request.send_to_beluga!
CareValidate.retry_via_beluga(contact: ::Ontraport::Meta::Contact.get_by_id(script.contact))
```

Lo que hacen es:
1. cambiar el estado del request a `routed_to_beluga`
2. cambiar el prescriber del contacto en Ontraport a "Beluga"

Se corre el job en sincrono para tener una ejecución inmediata. Una vez revisar que:

- El request haya pasado a `routed_to_beluga`
- El script tenga un valor en Master ID
- Preguntar a CS si se creó una visita en Beluga


## Casos de Starter Pack en Ontraport 🟡ℹ️

Etiquetas: #om_starter_pack_ontraport

Estos dos casos:

- OM-11394
- OM-11450

### Caso OM-11394

Para el caso OM-11394 Fabian me explicó que había que revisar las respuestas del Checkin en Ontraport. Particular énfasis en la respuesta a la pregunta _When did you last take your prescribed weight loss medication?_.

En este caso, el cx respondió _Within past month_. Según las reglas, ==el Starter Pack se activa cuando el CX pasó más de dos meses desde la última vez que el cx tomó la medicina.==

Según la IA de Slack:
> if the customer says it's been more than 1 month (but less than 2), the Starter Pack option won't be enabled through the normal check-in flow, and per policy agents shouldn't coach them to pick the 2-month answer if that's not actually true.


#### Solución para OM-11394

Fabian me indicó estos pasos que debe seguir CS y luego yo:

1. First check if the current case has been put in on hold.
	1. Copia a Sarah Gray si es necesario
2. Ask to CS to add credits/refund to cx account. Tier 3 is not able to resubmit this request for stater because this depends on cx checkin answers.
3. Reset Checkin and ask cx to complete a new one.

El paso 3 es el que yo completo. Para completarlo debo ir al Script en Ontraport y activar los Automations:

- [LIVE] Reset Script
- [LIVE CLICKFIX] URL Generation Redirect - Cookie

Hacer un impersonate y revisar que la pregunta de los meses se pueda responder.

### Caso OM-11450

En este el cx sí contestó como se espera para que se habilite el Starter Pack. Respondió _It has been over 2 months_. Así que procedí y apliqué los Automations en Ontraport. Vi que se pudo acceder de nuevo al Checkin pero tenía respuestas seleccionadas.

Informé al equipo de Tier 2. Estaré esperando...

## Caso OM-11464 - Stuck en PrescriptionWritten y PerfectRx 🟡ℹ️

Etiquetas: #om_stuck_in_prescription_written #om_perfect_rx

Típico caso de Member Period que no pasa de PrescriptionWritten. Hice ResubmitToMSO y se quedó pegado de nuevo. Corrí el script de debuggeo y hubo que actualizar la rama de PerfectRx. Al actualizar obtuve esto:
```
--> patient linked but never transmitted. CreatePrescriptionJob is the transmitter.
    prescriber_name: "Beluga Health" | practice_id: resolvable
```

Revisando más con Claudio llegamos al punto donde se corrió esto:
```ruby
client = PerfectRx::Client.new(config: PerfectRx.config, logger: Rails.logger)
PerfectRx::FetchPatient.by_external_nk!(client:, nk: "019307c8-4887-7ca3-aed1-5ffd4cdf64b9")
```

Que nos dio el mensaje:
```
app/services/perfect_rx/fetch_patient.rb:60:in 'PerfectRx::FetchPatient.handle_patient_result!': Could not find patient with that ID in system. (PerfectRx::FetchPatient::ApiError)
```

## Caso OM-11447 - Resubmit de WeightLossFollowup a WeightLoss 🟡ℹ️

Etiquetas: #om_salesforce_resubmit_to_weightloss

Dice el reporte:
> beluga merge chart issue and now they want this resubmitted as weightloss instead of weightlossfollowup on SF

En Ontraport eso se logra de otra forma así que ni idea. Le pregunté a Jaime pero no sabía tampoco. Así que le seguí la corriente a Claudio.

Dijo que había que hacer esto:
```ruby
mp   = Salesforce::Account.find_by(person_email: "CorreodelCX").latest_member_period
pend = mp.clinical_encounters.where(status: "Pending").order(:startdate).last

# esperado: true
pend.med_picker_recommendation.id == mp.latest_med_picker_recommendation.id

original = mp.slice(:customer_lifecycle_stage, :loyalty_points)
forced   = {customer_lifecycle_stage: "NewNonTransfer"}
forced[:loyalty_points] = "Zero" if mp.customer_type.in?(%w[B2C B2B])

begin
  mp.update!(forced)
  
  # imprime anterior vs nuevo y espera 'y'
  Salesforce::ResubmitToMso.call(clinical_encounter: pend)
ensure
  mp.reload.update!(original)
end

new_ce = mp.reload.clinical_encounters.order(:startdate).last
new_ce.slice(:id, :status, :visit_type, :source_system_identifier)
```

Explico lo que pasa:
- Obtenemos el Member Period y el Clinical Encounter en "Pending"
- Comprobamos el estado de la recomendación de MedPicker
- Copiamos los valores `customer_lifecycle_stage` y `loyalty_points`
	- Se copian porque necesitan restaurarse al final
- Se preparan los nuevos valores para esos dos campos:
	- `NewNonTransfer`
	- y `Zero` para loyalty points
- En el bloque begin/ensure:
	- Se actualiza el Member Period con los valores modificados
	- Se ejecuta un ResubmitToMso para el Clinical Encounter anterior (en pending)
	- Cuando se complete, se restaura el Member Period con los valores originales
- En la verificación:
	- Se busca el Clinical Encounter más reciente y se comprueba que salgan los valores que nos interesan:

```ruby
{"id" => "01a0b643-edde-73fe-bc26-69353973df61", "status" => "Pending", "visit_type" => "WeightLoss", "source_system_identifier" => "01A0B643-EDEF-7FF9-BD9A-283DD82D86DA"}
```


Al verificar:
- Se creó un nuevo Clinical Encounter en estado `Scheduled` y con Service Type "Weight Loss"
- El Clinical Encounter anterior se canceló
- El Member Period estaba trabado en "Ready to Create Visit" y pasó a "Visit Created"
- El Member Period volvió a sus valores originales

La verificación es con:
```ruby
mp.reload.slice(:status, :customer_lifecycle_stage, :loyalty_points)
=> {"status" => "VisitCreated", "customer_lifecycle_stage" => "Existing", "loyalty_points" => "Three"}

pend.reload.status
=> "Cancelled"

mp.latest_med_picker_recommendation.slice(:id, :status)
=> {"id" => "01a0b643-d50a-77f3-9f53-b577ddb75184", "status" => "RecommendationMade"}
```

## Caso OM-11496 - Stuck in Submitted migrado a Salesforce 🟢ℹ️

Etiquetas: #om_stuck_in_submitted #om_migration_op_to_sf 

Típico caso de cx migrado de Salesforce a Ontraport pero con un Script en curso. El Script quedó trabado en Submitted pero cuando fui a revisar el cx ya había sido migrado a Salesforce.

La forma que Jaime me explicó proceder aquí es:
- Limpiar el campo `salesforce_account_nk` del Account
- En Ontraport, deschulear el campo "Migrated To Salesforce?"
	- O correr el comando desde consola cambiando a false:

```ruby
contact = Ontraport::Meta::Contact.get_by_id(ontraport_contact_id)
contact.update_all(migrated_to_salesforce: 1) # Set "Migrated To Salesforce?" to true
```

Después de eso se puede hacer el resubmit normal.

> [!Note]
> Tengo dudas de si esto es necesario ya que vi que el Script pasó "Order at Pharmacy" sin yo hacer nada.