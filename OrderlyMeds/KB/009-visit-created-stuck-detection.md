# Estado `VisitCreated` y detección de casos atascados

## Qué significa

`VisitCreated` es un estado de la state machine de [[001-member-period|Member Period]] (`Salesforce::MemberPeriod`, `app/models/salesforce/member_period.rb`). Significa que la visita (telesalud/async) ya fue enviada al proveedor médico (Beluga Health o CareValidate) y **está esperando la decisión del prescriptor** (prescribir o referir/rechazar).

## Cómo se entra y se sale del estado

**Entrada:**
- `ReadyToCreateVisit → VisitCreated` vía el evento `create_visit!`, disparado desde `Patient::Checkin::ClinicalEncounterScheduler#confirm` (`app/services/patient/checkin/clinical_encounter_scheduler.rb:69-78`).
- La visita se envía al proveedor externo mediante los jobs de Sidekiq (guardados por `VisitCreationGuard`, `app/sidekiq/concerns/visit_creation_guard.rb`):
  - `BelugaHealth::Scheduler::CreateVisitJob` (queue `within_5_minutes`)
  - `CareValidate::Scheduler::CreateVisitJob` (queue `within_5_minutes`, `retry: 18`)
- `OrderCanceledByPharmacy → VisitCreated` vía `recreate_visit_after_pharmacy_cancel` (reintento tras cancelación de farmacia).

**Salida normal:** llega un webhook del proveedor con el resultado de la revisión:
- Beluga: `BelugaHealth::Webhooks::ConsultConcluded` (`lib/beluga_health/webhooks/consult_concluded.rb`)
- CareValidate: `CareValidate::ApproveRequest` / `CareValidate::RejectRequest` (`app/services/care_validate/{approve,reject}_request.rb`) — ver también [[008-state-machine-care-validate-request]]

Ambos llaman a `Patient::Visit.record_outcome` (`app/services/patient/visit.rb:16-51`), que exige `member_period.visit_created?` y transiciona:
- Prescrito → `clinical_encounter.status = "Finished"` + `member_period.complete_visit!` → `VisitCompleted`
- Referido → `clinical_encounter.status = "Referred"` + `member_period.refer!` → `Referred`

## ¿Cuánto tiempo debería durar?

Es tiempo de espera por revisión humana de un prescriptor licenciado, **no** un timeout técnico. El propio sistema define el SLA en `app/services/member_periods/stuck_detection.rb:3`:

```ruby
VISIT_CREATED_STUCK_HOURS = 48
```

Unas horas de espera dentro del mismo día hábil son normales. Pasadas **48 horas**, el sistema lo marca oficialmente como "atascado" (`stuck`).

## Detección y alertado automático

- **`MemberPeriods::StuckDetection`** (`app/services/member_periods/stuck_detection.rb`) — calcula `stuck_since` a partir de la última transición hacia `VisitCreated` (vía `Salesforce::MemberPeriodHistory.latest_transitions_into`).
- **`MemberPeriods::StuckMemberPeriodsReportJob`** — cron registrado en `config/initializers/sidekiq.rb:65`, corre **7:00 AM Eastern, lunes a viernes**. Revisa todos los `MemberPeriod` en `VisitCreated` (y `PharmacyOrderConfirmed`) de la línea de servicio `WeightlossAntiagingMetabolicSupport`, emite métricas (`member_periods.stuck.visit_created`) y encola `ReportStuckMemberPeriodJob` por cada caso atascado.
- **`MemberPeriods::ReportStuckMemberPeriodJob` → `Linear::CreateMemberPeriodDelayTask`** (`app/services/linear/create_member_period_delay_task.rb`) — crea automáticamente un ticket en Linear (equipo Support Engineering, asignado a Briggs Garland), deduplicado por `fingerprint_key: member_period.sfid`. El texto del diagnóstico (`app/services/member_periods/report_stuck_member_period_service.rb:32-39`):

  > "The member period is waiting for a provider to review and approve the visit. Ask Beluga about the visit status and check if resubmit to MSO is required."

- Visibilidad en admin: `Admin::DelayedMemberPeriodsController`, vista `admin/delayed_member_periods/index.html.erb`, query `MemberPeriods::DelayedMemberPeriodsQuery`.
- Hay además una caché de 30 min (`MemberPeriods::StuckMemberPeriodService`) usada para un banner en tiempo real en el dashboard del paciente/admin — es un chequeo distinto al cron diario.

⚠️ El cron solo corre en días hábiles: algo atascado un viernes por la tarde no genera ticket hasta el lunes.

## Qué revisar si un caso lleva varias horas atascado

1. **¿El `CreateVisitJob` tuvo éxito?** Revisar reintentos/fallos en Sidekiq (el job de CareValidate reintenta hasta 18 veces sin que la visita llegue a enviarse realmente).
2. **Estado del caso directamente en Beluga/CareValidate** — es lo primero que sugiere el propio ticket automático.
3. **¿Existe un `IncomingWebhook` de conclusión de consulta y se procesó sin error?** `record_outcome` lanza excepción dura si el estado no es el esperado (`"Cannot record outcome of member period with status #{member_period.status}"`).
4. **Remediación manual:** `Salesforce::ResubmitToMso` (`app/services/salesforce/resubmit_to_mso.rb`) — cancela el encuentro clínico atascado, resetea `member_period.status` a `ReadyToCreateVisit` y reagenda un nuevo encuentro. Se invoca desde `Admin::CareValidateController` / `MonolithApi::V1::ResubmitToMsoController`. Es el "resubmit to MSO" que menciona el ticket automático.
5. Antes de escalar manualmente, buscar en Linear un ticket `[AUTO] Stuck on VisitCreated - <account>` ya existente.
