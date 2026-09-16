# OM Ciclo 55

> [!Info]
> Del Jueves 10 al Miércoles 23 de Septiembre.

## Caso OM-11347 - MP Stuck at ReadyToCreateVisit 🟢

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


## Caso OM-11427 - Reroute to Beluga ℹ️

Etiquetas: #om_reroute_to_beluga 

Necesito cambiar el prescriber de un CX a Beluga y hacer el resubmit.

Hay que hacer esto:
```ruby
script = ::Ontraport::Meta::Script.get_by_id(950674)
request = CareValidate::Request.find("01a082b8-e322-799a-bea5-8bc8d40a49c2")
icwhs = IncomingWebhook.where(id: request.incoming_webhook_ids)
wh = icwhs.first
wh.update!(state: "pending")

request.send_to_beluga!
CareValidate.retry_via_beluga(contact: ::Ontraport::Meta::Contact.get_by_id(script.contact))
ProcessIncomingWebhookJob.new.perform(wh.id)
```

- Ubicar el script para poder cambiar el prescriber del Contacto en Ontraport
- Ubicar la Request de Care Validate
	- Y el webhook que sea de tipo "ontraport"
- Cambiar el estado del webhook a "pending"

Luego estos:
```ruby
request.send_to_beluga!
CareValidate.retry_via_beluga(contact: ::Ontraport::Meta::Contact.get_by_id(script.contact))
```

Lo que hacen es:

1. cambiar el estado del request a `routed_to_beluga`
2. cambiar el prescriber del contacto en Ontraport a "Beluga"

Se corre el job en sincrono para tener una ejecución inmediata.