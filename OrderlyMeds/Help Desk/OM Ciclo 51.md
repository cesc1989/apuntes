# OM Ciclo 51

> [!Info]
> Del Jueves 16 de Julio al Miércoles 29 de Julio.

## Caso OM-10057 - Unable to Recommend Script 🟡

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

En el CV::Request que cancelé el MedId Requested fue OH para Evoluciona:
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