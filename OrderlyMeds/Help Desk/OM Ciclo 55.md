# OM Ciclo 55

> [!Info]
> Del Jueves 10 al Miércoles 23 de Septiembre.

## Caso OM-11347 - MP Stuck at ReadyToCreateVisit 🟢

Etiquetas: #om_stuck_in_ready_to_create_visit

Casos con el mismo problema (no necesariamente misma solución):

- OM-9396 -> [[OM Ciclo 48#Caso OM-9361 - MP Stuck in ReadyToCreateVisit 🟢]]
- OM-7070

### Paso 1: intentar crear la visita

Esto es lo que sugiere el Notion:
```ruby
mp = Salesforce::MemberPeriod.find_by(omid: "019fe9b0-61ae-7e3a-b805-15b3df930ac2")
ce = mp.clinical_encounters.last
BelugaHealth::Scheduler::CreateVisitJob.new.perform(ce.id)
```

Sin embargo, el MP de este caso es de Care Validate. Hice resubmit normal.

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

## Caso OM-11414 - Stuck in ReadyToCreateVisit de Beluga 🟡

Etiquetas: #om_stuck_in_ready_to_create_visit

Al contrario que OM-11347 al inicio de este doc, este como sí es de Beluga intenté el troubleshoot pero dio el mismo problema del status 400.

Así que mandé el mensaje tal cual hice en OM-9396 -> [[OM Ciclo 48#Caso OM-9361 - MP Stuck in ReadyToCreateVisit 🟢]]

Mensaje:
> When trying to create the visit, Beluga returns the error: _"Status 400: Patient not eligible for this visit."_ Please contact the provider to determine why the patient is not eligible for the visit.