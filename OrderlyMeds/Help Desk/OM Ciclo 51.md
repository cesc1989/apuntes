# OM Ciclo 51

> [!Info]
> Del Jueves 16 de Julio al Miércoles 29 de Julio.

## Caso OM-10057 - Unable to Recommend Script 🟢

Etiquetas: #om_unable_to_recommend_script

Dice:
> OMFS data shows:
> Unable to recommend script: Unknown reason

Similares:

- OM-10109
- OM-7655
- OM-9156 -> Fabian deja un buen comentario.

### Caso relacionado OM-9156 ℹ️

Fabian dice:
> We are unable to recommend an alternative script at this time because both B6 and B12 were selected, and there are currently no medication options available without at least one of those ingredients.
>
> Please cancel the order and issue a refund. Then, ask the customer whether they are willing to receive a medication that includes either B6 or B12. If so, they can complete a new checkin and we can proceed with a new recommendation.

### Sensitivities: B6 y B12

Este caso también ambas están relacionadas:
![[om_10057.png]]

¿Qué pasa con eso?

## Caso OM-10167 - Error de MedPicker 🟡

Etiquetas: #om_care_validate_error #om_medpicker_issue #om_wrong_pharmacy

El típico caso del problema que el MedPicker ofrece una farmacia en el estado equivocado.

En el `CareValidate::Request` que cancelé el MedId Requested fue OH para Evoluciona:
```
nwDdccjIgYP0Rpdl1UKC2eDSg7jcDTjk
```

Sin embargo, en el Prescribed dio este en ND para CasaPharmaRx
```
LZDRgy7jShPoP6d3yfqU8PUeww4f2iEi
```

Luego del resubmit tiraba el mismo MedId Requested pero el estado era ND. Lo cual era raro. Probé con el botón "Fix Medpicker Selection" pero no pasó nada.

Luego probé con "Resubmit Latest Ontraport Webhook" y ese sí lo cambió a otro MedId en OH para Evoluciona:
```
tW3hmw2j6faAtjLD7jYAWhKLOHV61Jmz
```

## Casos de Script Error: No matching recommendations 🟡

Etiquetas: #om_no_matching_recommendations 

OM-10139 dice:
> Rx wasn't process due to an error. OMFS data shows: "No matching recommendations for these patient preferences."

Y OM-10092 dice:
> Rx wasn't forwarded due to an error. OMFS data shows "No matching recommendations for these patient preferences."

Iba a hacer lo que describe [[OM Ciclo 50#Caso OM-9790 - Script error con needs_requested_medpicker_data 🟢ℹ️]] pero al revisar los Scripts veo que están en "Order at Pharmacy". Pensé que los encontraría en "Script Error".

**Actualizaciones**
- No hice nada y el Script de 10139 el Viernes estaba en "Order at Pharmacy"
	- Ya hoy, 21 de Julio, estaba en Shipped
- El 10092 sigue en Order at Pharmacy pero no se ha enviado aún

## Caso OM-10105 - Missing Orders después de Import de OP a SF 🟢

Etiquetas: #om_missing_orders #om_migration_op_to_sf

El reporte decía que no se veían las ordenes de un cx. Así se veía al impersonar:
![[om_10105.00.png]]

El Cliente tuvo su orden completa en Ontraport y cuando revisé el caso ya estaba en Salesforce. Algo quedó mal durante la migración.

Después una revisión exhaustiva con Claudio me di cuenta el detalle. El Member Period que correspondía con el Script con Outcome "Delivered" no tenía dicho valor en el campo `ontraport_imported_outcome`.

Esto hacía que el llamado a la función `is_a_valid_order?` en `Patient::Connectors::Salesforce::Script` no devolviera el valor correcto para enlazar dicha `Salesforce::Order` con la UI.

### Las funciones y el MP en estado ImportedFromOntraport

La función `is_a_valid_order?` en la clase en `app/models/patient/connectors/salesforce/script.rb`:
```ruby
def is_a_valid_order?
	if imported_from_ontraport?
		valid_shipped_order_from_ontraport?
	else
		true
	end
end
```

Por su parte `valid_shipped_order_from_ontraport?` hace una comparación del Outcome del Script para que sea "Shipped".

El Member Period tiene estado `ImportedFromOntraport` así que la función entra al condicional de `imported_from_ontraport?` y ejecuta el código de esa otra función:
```ruby
def valid_shipped_order_from_ontraport?
	outcome == "Shipped"
end
```

`outcome` es una función:
```ruby
def outcome
	return nil unless member_period
	@outcome ||= Patient::Connectors::Salesforce::OrderStatus.new(member_period_status: member_period.status, ontraport_outcome: member_period.ontraport_imported_outcome).get_outcome
end
```

Aquí es donde estuvo la solución. Cuando revisé `member_period.ontraport_imported_outcome` me daba nulo. Ahí supe que tenía que corregir ese campo.

#### Actualizar ontraport_imported_outcome

Quería probar si con actualizar:
```ruby
mp.update!(ontraport_imported_outcome: "Delivered)
```

Sería suficiente así que probé la salida de lo que pasa en `get_outcome` de `app/models/patient/connectors/salesforce/order_status.rb`. Vi este llamado:
```ruby
outcome = Patient::Connectors::Ontraport::OrderStatus.new(outcome: ontraport_outcome).name if ontraport_outcome.present?
```

Y probé entonces:
```ruby
Patient::Connectors::Ontraport::OrderStatus.new(outcome: "Delivered").name
=> :medication_delivered
```

Ahí comprobé que esa sería la solución ya que ese era un valor que `get_outcome` tomaría como bueno para lograr el objetivo de `valid_shipped_order_from_ontraport?`.

### La solución: actualizar el campo

Así:
```ruby
member_period_omid = "019ef178-5c50-7c0a-82b4-099ed78acefe"
correct_outcome = "Delivered"
mp = Salesforce::MemberPeriod.find_by(omid__c: member_period_omid)

mp.update!(ontraport_imported_outcome: correct_outcome)
```

Se refleja enseguida en la UI de Salesforce:
![[om_10105.02.png]]

Una vez hecho eso se puede ver la orden en la lista en el portal del cliente.
![[om_10105.01.png]]

## Caso OM-10203 - CV Request stuck en needs_prescriber_submission 🟢

Etiquetas: #om_needs_prescriber_submission

> [!Note]
> Otro caso como este fue el OM-9981.

Típico caso de hacer un resubmit a un Script en Ontraport que se quedó en "Submitted". Luego de hacer eso el `CareValidate::Request` se queda en `needs_prescriber_submission` y no progresa.

Al parecer el problema es que este nuevo ni el anterior tienen un valor en `case_nk`:
![[om_10203.01.png]]

Comparado con lo mismo para otro CX el cual sí progresó normalmente el resubmit:
![[om_10203.02.png]]

> Causa raíz: Este paciente **nunca tuvo un caso creado en CareValidate**. El primer request fue cancelado sin `case_nk`, y el segundo llegó como `care_validate_checkin` pero no hay caso al cual hacer checkin. El `SendCheckinJob` falló silenciosamente con `CaseIdNotFound`.

### La Solución: CareValidate::FindOrCreateCaseJob

La solución fue identificar los valores necesarios para correr el job `CareValidate::FindOrCreateCaseJob`:
```ruby
request = CareValidate::Request.find("ID")

CareValidate::FindOrCreateCaseJob.new.perform(request.incoming_webhook_ids.first, request.id, request.script_nk)
```

Cuando terminó la ejecución el request ya tenía valores en los campos `nk` y `case_nk`:
```ruby
nk: "948efbc9-7190-43c3-9fa7-636863ae3716",
case_nk: "3b83fc1e-006b-49ed-9b07-3e10bc8f1144",
```

Y ya se aprecia en la UI:
![[om_10203.03.png]]

## Caso OM-10121 - connect ECONNREFUSED 🟡

Etiquetas: #om_care_validate_error #om_connection_refused

El error se ve es en el portal de Care Validate:
```
connect ECONNREFUSED 3.143.37.86:443
```

Y se ve así:
![[om_10121.png]]

Tiene un caso similar que es el OM-9927. Vi que Fabian hizo un resubmit así que hice lo mismo para este.

### Cambios de Estado del Member Period después del resubmit

Estos son todos los cambios que tuvo el campo status para llegar hasta donde nos interesa.

- Después del resubmit pasó a VisitCreated
	- Se quedó ahí varios días
- El 17 de Julio hizo todos estos cambios:
	- 7/17/2026, 2:57 PM: pasó a Visit Completed
	- 7/17/2026, 3:27 PM: pasó a Prescription Written
		- Después pasó a Waiting On Pharmacy Confirmation
		- Y finalmente a Pharmacy Order Confirmed