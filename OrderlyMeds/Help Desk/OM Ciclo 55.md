# OM Ciclo 55

> [!Info]
> Del Jueves 10 al Miércoles 23 de Septiembre.

## Caso OM-11347 🟡

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