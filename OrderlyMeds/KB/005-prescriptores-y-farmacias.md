# Prescriptores y Farmacias: Beluga, CareValidate, SmartPharma, PerfectRx

## Los cuatro sistemas ocupan dos roles distintos

```
[PRESCRIPTORES]          [FARMACIAS]
Beluga Health    ──┐
                   ├──> SmartPharma
CareValidate     ──┤    PerfectRx
                   └──> Casa/Evoluciona
```

La farmacia destino la determina el `remotePharmacyIdentifierNk` que devuelve MedPicker al momento de la prescripción — es independiente de si fue Beluga o CareValidate quien escribió el Rx.

---

## Prescriptores (quiénes escriben el Rx)

### Beluga Health

- **Qué es**: Plataforma de telesalud / prescripción. Es la integración más antigua (`DEFAULT_NAME` del modelo `Prescriber`).
- **Cómo funciona**: OrderlyMeds envía un "visit form" (cuestionario + fotos: ID, selfie, Rx actual) a Beluga vía API. Un proveedor de Beluga revisa el caso, posiblemente hace una videoconsulta, y dispara webhooks de vuelta.
- **Tipos de servicio**: `WeightLoss`, `WeightLossFollowup`, `WeightlossAntiagingMetabolicSupport`
- **Webhook principal**: `prescription_written` → crea un `RxWrittenBundle` → crea la `Order`

Webhooks que maneja:

| Evento | Efecto |
|--------|--------|
| `prescription_written` | Crea/linkea un `RxWrittenBundle`, agenda `CloseRxWrittenBundleJob` |
| `consult_concluded` | Registra resultado de visita (prescribed/referred) |
| `consult_canceled` | Reenvía a cola OMFS |
| `chat_message` | Entrega mensaje al paciente |
| `customer_service_message` | Postea nota en Ontraport |
| `booking` | Registra scheduling (created/rescheduled/cancelled/no-show); Slack alerts para no-shows |

### CareValidate

- **Qué es**: Plataforma más nueva de telesalud con flujo estructurado de aprobación de casos. Foco exclusivo en GLP-1.
- **Cómo funciona**: Se envía un "dynamic case form". El proveedor revisa y devuelve un webhook `ADD_CASE_DECISION` con la decisión (APPROVED / REJECTED / NO_DECISION), el MedID prescrito, dosis y credenciales del prescriptor.
- **Tipos de servicio**: Solo `GLP1`

**Diferencia clave vs Beluga — capa de validación interna**: Cuando CareValidate aprueba, OrderlyMeds NO acepta la decisión ciegamente. Corre 5 validaciones antes de continuar:

1. `PrescribedMedIdExists` — el MedID existe en el catálogo de MedPicker
2. `ContraindicationsMatch` — no hay discrepancia de contraindicaciones
3. `BaseMedMatch` — la clase base del medicamento coincide con lo solicitado
4. `NumberOfWeeksMatch` — el supply de semanas está alineado
5. `KnownMedIds` — el MedID está en la lista de válidos conocidos

Si todas pasan → estado `written`. Si alguna falla → `flagged_for_review` + alerta en Slack.

**Fallback a Beluga**: Si CareValidate falla al crear el caso ("email already exists in another organisation" o error de fotos), `CareValidate::RerouteToBeluga` cancela el caso en CareValidate y reencamina al paciente a Beluga automáticamente. El state machine de `CareValidate::Request` tiene un estado `routed_to_beluga` para este path.

**Peculiaridad**: CareValidate también tuneliza webhooks de órdenes de Casa (`ORDER_CREATED`, `ORDER_STATUS_CHANGED`, `ORDER_TRACKING_ADDED`) a través de su mismo endpoint.

Webhooks que maneja:

| Evento | Efecto |
|--------|--------|
| `ADD_CASE_DECISION` | Pipeline de validación → aprueba o flagea para revisión |
| `ADD_CASE_COMMENT` | Mensaje de proveedor al paciente |
| `DECISION_DELETED` | Maneja retractación de una decisión previa |
| `ORDER_CREATED` | Delega a `Casa::OrderCreatedJob` |
| `ORDER_STATUS_CHANGED` | Delega a `Casa::OrderStatusChangedJob` |
| `ORDER_TRACKING_ADDED` | Delega a `Casa::Webhooks::OrderTrackingAdded` |

---

## Farmacias (quiénes dispensan el medicamento)

### SmartPharma

- **Qué es**: Farmacia de **compounding** (preparados magistrales) sobre la plataforma Pharmetika. Usa recursos FHIR.
- **Qué dispensa**: Solo medicamentos compuestos:
  - Semaglutide compuesto (ej: `Semaglutide Gastro 5mg/ml 10MG/2ML`)
  - Tirzepatide compuesto (ej: `Tirzepatide Gastro 30mg/ml 60MG/2ML`)
- **`reason_for_compounding`**: "Concentration Adjustment Necessary" (código 6) — lenguaje regulatorio estándar de compounding.
- **Modelo técnico**: **Medication Order** — flujo multi-paso:
  1. Crear paciente en Pharmetika
  2. Enviar alergias (FHIR AllergyIntolerance)
  3. Validar orden (`PUT .../validate`)
  4. Enviar orden (`PUT .../submit`)
  5. Actualizar CRM
- **Autenticación**: Token-based (cacheado como `"smart_pharma_token"`).
- **Feature flag**: `PROCESS_SMART_PHARMA_ORDERS`

Webhooks que recibe:

| Evento | Efecto |
|--------|--------|
| `shipment_created` | Marca orden como enviada, guarda tracking |
| `shipment_delivered` | Actualiza timestamp de entrega |

### PerfectRx

- **Qué es**: Farmacia sobre la plataforma SmartScripts.
- **Qué dispensa**: Más amplio que SmartPharma — compuestos **y** medicamentos de marca:
  - Compuestos: Semaglutide 2.5mg/ml, Tirzepatide 18mg/ml, etc.
  - De marca: Zepbound, Wegovy, Mounjaro
- **Modelo técnico**: **eScript transmission** (`POST /v1/prescriptions/transmitEScript`) — similar a cómo las farmacias tradicionales reciben recetas electrónicas. Más directo que SmartPharma (no tiene el flujo multi-paso de validate→submit).
- **Autenticación**: JWT-based (cacheado como `"perfect_rx_jwt"`).
- **Feature flag**: `PERFECT_RX_CREATE_PRESCRIPTION`
- **Detalle por prescriptor**: `PerfectRx::Config#practice_id` retorna diferentes `practice_id` según si el prescriptor fue Beluga o CareValidate. `CreatePrescription#encode_strength` también tiene ramas por prescriptor para Zepbound.

Webhooks que recibe:

| Evento | Efecto |
|--------|--------|
| `patient_creation_success` | Guarda `smart_scripts_patient_nk`; dispara procesamiento de prescripción |
| `patient_creation_failure` | Error/alerta |
| `order_received` | Marca `received_at` |
| `order_shipped` | Guarda `shipped_at`, carrier, tracking, `smart_scripts_rx_number` |
| `order_canceled` | Marca prescripción como cancelada |
| `patient_address_updated` | Sincroniza dirección actualizada |
| `patient_updated` | Sincroniza demografía del paciente |

---

## Resumen comparativo

| | Beluga Health | CareValidate | SmartPharma | PerfectRx |
|--|--|--|--|--|
| **Rol** | Prescriptor | Prescriptor | Farmacia | Farmacia |
| **Tipo servicio** | Weight Loss (amplio) | GLP-1 solo | Compounding | Compounding + Marca |
| **Validación extra** | No | Sí (5 checks) | — | — |
| **Plataforma** | Propia | Propia | Pharmetika | SmartScripts |
| **Modelo API** | Visit form + webhooks | Dynamic case + webhooks | FHIR Medication Order | eScript |
| **Autenticación** | — | — | Token | JWT |
| **Feature flag** | — | — | `PROCESS_SMART_PHARMA_ORDERS` | `PERFECT_RX_CREATE_PRESCRIPTION` |

---

## Interacciones entre sistemas

- **Ambos prescriptores pueden alimentar ambas farmacias**: el routing lo decide MedPicker, no el prescriptor.
- **CareValidate puede fallar y redirigir a Beluga**: `CareValidate::RerouteToBeluga`.
- **SmartPharma registra métricas por prescriptor**: `Metrics.increment("smart_pharma.order.beluga")` / `Metrics.increment("smart_pharma.order.care_validate")`.
- **`BelugaHealth::PrescriptionWritten` lleva un `patient_external_nk`**: es el `PerfectRx::Patient.external_nk` — acopla el registro del prescriptor con el de la farmacia desde el momento en que se crea el Rx.
