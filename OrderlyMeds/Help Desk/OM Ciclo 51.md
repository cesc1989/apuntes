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

Iba a hacer lo que describe [[OM Ciclo 50#Caso OM-9790 - Script error con needs_requested_medpicker_data 🟢ℹ️]]

