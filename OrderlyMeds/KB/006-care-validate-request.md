# CareValidate::Request — Estados y Flujo

## Flujo principal (happy path)

```
needs_requested_medpicker_data   ← estado inicial
         │
         │ ready_for_care_validate_submission
         ▼
needs_prescriber_submission
         │
         │ ready_for_crm_sync
         ▼
needs_crm_update
         │
         │ waiting_for_prescription
         ▼
waiting_for_prescription          ← espera webhook ADD_CASE_DECISION de Care Validate
         │
         │ needs_review (webhook recibido)
         ▼
needs_review
         │
         │ received_prescription (aprobación automática validada)
         ▼
written                           ← estado final exitoso
```

## Flujos alternativos

| Evento | Desde | Hacia |
|---|---|---|
| `send_to_beluga` | `needs_requested_medpicker_data`, `needs_prescriber_submission`, `needs_crm_update` | `routed_to_beluga` |
| `reset_to_initial_state` | cualquier estado | `needs_requested_medpicker_data` |
| `manually_resolve` | `needs_requested_medpicker_data`, `waiting_for_prescription` | `manually_resolved` |
| `cancel` | cualquier estado | `canceled` |
| `mark_needs_resubmission` | `written` | `needs_resubmission` |

## Estados terminales / excepcionales

- **`routed_to_beluga`** — desviado a Beluga Health (otra farmacia)
- **`manually_resolved`** — resuelto manualmente por admin
- **`canceled`** — cancelado (desde cualquier estado)
- **`needs_resubmission`** — requiere reenvío (solo desde `written`)

### Qué pasa al hacer resubmit (`mark_needs_resubmission`)

El evento `mark_needs_resubmission` en sí **no tiene efectos secundarios** — es un simple cambio de estado en DB (no hay `before_transition`/`after_transition` en este state machine). No dispara ningún job ni llamada HTTP en el momento en que se ejecuta. Se dispara desde el botón "Resubmit" del panel de admin (`Admin::OverviewController#resubmit_care_validate`).

La resubmisión real a Care Validate ocurre **después**, cuando llega el próximo webhook de checkin de Ontraport:

1. `CareValidate::FindOrCreateRequestByScriptId` detecta el request en `needs_resubmission`, lo **cancela** (conservando su `case_id`) y crea un **`CareValidate::Request` nuevo** que hereda ese `case_id`.
2. El nuevo request recorre el pipeline normal (`ProcessRequestJob` → `GetMedpickerDataJob` → ...) hasta volver a `needs_prescriber_submission`/`needs_crm_update`.
3. Ahí `CareValidate::SendCheckinJob` sí manda algo nuevo a Care Validate:
   - **GET** `/api/v1/customer-latest-case-id` (recupera el case id existente).
   - **POST** `/api/v1/customer-dynamic-case-form` (nuevo form response/checkin dentro del mismo case de CV).
   - (Rama análoga en `FindOrCreateCaseJob` para `initial_signup`, con `formTitle: "Resubmission - OrderlyMeds GLP Intake Form"`.)

También existe `CareValidate::FixResubmitOntraportWebhook`, una utilidad de ops que hace lo mismo de forma inmediata (llama `SendCheckinJob.perform_async` directo) sin esperar el próximo webhook.

**En resumen:** el cambio de estado por sí solo no toca Care Validate, pero el flujo de resubmit completo sí termina generando una llamada saliente nueva a la API de Care Validate, disparada por el próximo webhook de Ontraport.

## Tiempos en `waiting_for_prescription`

La salida de este estado es **event-driven** — ocurre cuando Care Validate envía el webhook `ADD_CASE_DECISION`. No hay timeout ni expiración automática.

| Umbral | Propósito |
|---|---|
| **48 horas** | Considerado "stuck" — aparece en reportes de Slack (`#ops_stuck_orders`) y panel de admin |
| **3 días** | `OntraportGapAnalyzer` lo marca como brecha en el pipeline |
| **7 días** | La UI del admin pone el contador de días en rojo |

El job `Orders::StuckOrdersReportJob` corre **diariamente a las 8am ET** y alerta en Slack sobre requests con más de 48 horas en este estado.

### Qué hacer cuando se supera el umbral de 48 horas

No existe lógica automática que saque el request de `waiting_for_prescription` ni que reintente el webhook — la resolución es siempre manual:

1. **Revisar la alerta** en `#ops_stuck_orders` o el panel de admin para identificar el request.
2. **Investigar por qué Care Validate no envió el webhook** `ADD_CASE_DECISION` (caso perdido, error en su lado, etc.).
3. Si no se puede resolver el caso vía Care Validate, usar la transición `manually_resolve` (disponible desde `waiting_for_prescription`) para llevarlo a `manually_resolved`.
4. Si el request sigue sin resolverse pasados 3 días, será marcado como brecha por `OntraportGapAnalyzer`; a los 7 días el contador en la UI de admin se pondrá en rojo, aumentando la visibilidad/urgencia.

## Notas

- El estado `pending` existe en el state machine pero no tiene transiciones activas — parece ser legacy.
- El state machine está definido en `app/models/care_validate/request.rb`.
- Los jobs que transicionan a `waiting_for_prescription`: `FindOrCreateCaseJob`, `SendCheckinJob`, `Scheduler::CreateVisitJob`.
