# OM Ciclo 56

> [!Info]
> Del Jueves 24 de Septiembre al Miércoles 7 de Octubre.

## Caso OM-11571 - Reiniciar Script para Starter Pack 🟡

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

Mismo caso de ordenes faltantes. Al Member Period le faltaba el Medication Request. Luego de reimportar seguía sin mostrarse la orden. Faltaba un OrderSummary por crear.

### Solución: reimportar OrderSummary

Con este código sacado y ajustado desde `Salesforce::OntraportAccountImporter`.
