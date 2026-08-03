# Member Period

## Concepto

Un **Member Period** es el ciclo completo de atención de un paciente en OrderlyMeds — representa una ronda de medicación desde el check-in hasta la entrega. Cada paciente puede tener múltiples member periods secuenciales a lo largo del tiempo.

- **Modelo:** `Salesforce::MemberPeriod` (`app/models/salesforce/member_period.rb`)
- **Tabla Salesforce:** `memberperiod__c`
- **Historial:** `Salesforce::MemberPeriodHistory` (`memberperiod__history`)

## Fechas clave

| Campo | Significado |
|---|---|
| `start_date` | Inicio del período (3 días después de confirmación de orden) |
| `end_date` | Fin del período de medicación |
| `checkin_due_date` | Cuándo debe hacer check-in el paciente |
| `checkin_deadline_date` | Fecha límite con gracia (`due_date` + 15 días) |
| `next_checkin_eligible_date` | Desde cuándo puede iniciar el próximo (`end_date` - 15 días) |

Constantes:
- `CHECKIN_GRACE_PERIOD` = 15 días
- `EARLY_CHECKIN_PERIOD` = 15 días
- `SHIPPING_TIME_ESTIMATE` = 3 días

## Estados (28 posibles)

Flujo principal:

```
InHealthScreening
  → HealthScreeningCompleted
  → ReadyForCheckin → CheckinDelayed → CheckinCompleted
  → ReadyForProductSelection → ReadyForOrderPayment
  → ReadyToCreateVisit → VisitCreated → VisitCompleted
  → PrescriptionWritten → WaitingOnPharmacyConfirmation
  → PharmacyOrderConfirmed → PharmacyOrderShipped
  → PharmacyOrderDelivered
```

- Desde casi cualquier estado se puede ir a `Canceled`
- Si se pierde el check-in → `MissedCheckin`
- `OrderCanceledByPharmacy` permite reiniciar desde `VisitCreated`
- Estado legacy: `ImportedFromOntraport`

## Atributos de clasificación

- **`customer_type`**: `Employee`, `B2C`, `B2B`
- **`customerlifecyclestage`**: `NewNonTransfer`, `NewTransferIn`, `Existing`, `Restarting`, `Cancelled`, `NotApplicable`
- **`loyaltypoints`**: `Zero` → `One` → ... → `Twelve` (avanza en cada período nuevo; empleados siempre `NotApplicable`)

## Relaciones principales

| Relación | Descripción |
|---|---|
| `account` | El paciente |
| `patient_checkin` | Check-in activo del período |
| `medication_requests` | Prescripciones escritas |
| `medication_dispenses` | Dispensados de medicación por la farmacia (ver abajo) |
| `orders` / `order_summaries` | Órdenes de compra |
| `shipments` | Envíos de farmacia |
| `clinical_encounters` | Visitas médicas |
| `med_picker_recommendations` | Sugerencias de medicación del algoritmo |
| `history` | Auditoría de cambios de estado |

## MedicationDispense

Modelo: `Salesforce::MedicationDispense` (`medicationdispense`). Representa el dispensado físico de un medicamento por la farmacia.

Se crea via `Salesforce::RecordMedicationDispense` (job de Sidekiq) cuando la farmacia confirma la recepción de la orden (webhooks de SmartPharma, PerfectRx, o CASA). Se vincula al `ClinicalEncounter` correspondiente.

**Estados posibles:** `Preparation`, `In-Progress`, `Cancelled`, `On-Hold`, `Completed`, `Entered-In-Error`, `Stopped`, `Declined`, `Unknown`

- `In-Progress` = la farmacia recibió la orden y está preparando la medicación → el MP debería estar en `PharmacyOrderConfirmed`
- Si la orden se cancela, todos los dispenses activos pasan a `Cancelled` automáticamente (vía `Orders::PostOrderCanceled`)

**Nota:** `mp.orders` devuelve `Salesforce::Order` (Draft/Activated), no el modelo local `Order`. El `Order` local (con `sent_to_pharmacy_at`, `prescription_written_at`, etc.) se vincula por `visit_identifier` → `OrderPrescriberRelation`.

## Automatización (Sidekiq Jobs)

| Job | Descripción |
|---|---|
| `CreateNewMemberPeriodsJob` | Crea el siguiente período cuando se entrega el actual |
| `CancelMissedMemberPeriodsJob` | Cancela períodos con check-in vencido |
| `MarkMemberPeriodsDeliveredJob` | Marca como entregados |
| `ScheduleVisitsForReadyMemberPeriodsJob` | Agenda visitas médicas |
| `StuckMemberPeriodsReportJob` | Reporta períodos atascados |

## Validaciones de negocio

- Clientes `Employee` deben tener `loyaltypoints = NotApplicable`
- Clientes `B2C`/`B2B` nuevos deben tener `loyaltypoints = Zero`
- Clientes `B2C`/`B2B` existentes no pueden tener `loyaltypoints = NotApplicable`
