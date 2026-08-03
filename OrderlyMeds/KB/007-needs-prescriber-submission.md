# Estado `needs_prescriber_submission` en `CareValidate::Request`

## Qué significa

Este estado indica que el request **ya tiene los datos de MedPicker listos** y está en espera de ser enviado a la plataforma CareValidate para que un prescriptor lo revise.

Es el punto intermedio entre:
- **Antes**: recopilar los datos de medicación del paciente (MedPicker)
- **Después**: crear o actualizar el caso en CareValidate para revisión médica

## Cómo llega a este estado

El evento `ready_for_care_validate_submission!` lo activa desde:

| Origen | Archivo |
|---|---|
| `GetMedpickerDataJob` (camino principal) | `app/sidekiq/care_validate/get_medpicker_data_job.rb:96` |
| `Scheduler::CreateVisitJob` (camino Salesforce) | `app/sidekiq/care_validate/scheduler/create_visit_job.rb:25` |
| `FixResubmitOntraportWebhook` (fix manual de ops) | `app/services/care_validate/fix_resubmit_ontraport_webhook.rb:40` |
| `FixMedpickerSelection` (fix manual de ops, escribe directo en DB) | `app/services/care_validate/fix_medpicker_selection.rb:28` |

## Qué pasa mientras está en este estado

`ProcessRequestJob` lo recoge y despacha al job correcto según el `source_event` del webhook entrante:

| `source_event` | Job | Acción |
|---|---|---|
| `initial_signup` | `FindOrCreateCaseJob` | Crea un nuevo caso en CareValidate |
| `care_validate_checkin` | `SendCheckinJob` | Envía un checkin a un caso existente |

Ambos jobs, al terminar exitosamente, llaman `ready_for_crm_sync!` y avanzan a `needs_crm_update`.

## Flujo resumido

```
needs_requested_medpicker_data
        │ GetMedpickerDataJob
        ▼
needs_prescriber_submission       ← este estado
        │ FindOrCreateCaseJob / SendCheckinJob
        ▼
needs_crm_update
        │
        ▼
waiting_for_prescription
```

## Errores silenciosos que dejan el request atascado aquí

Tanto `FindOrCreateCaseJob` como `SendCheckinJob` tienen manejo de errores que **retornan `nil` sin cambiar el estado** ni lanzar una excepción que Sidekiq pueda reintentar:

| Error | Comportamiento |
|---|---|
| `CareValidate::ProcessImageError` | Notifica Slack canal tech + `NotificationService`, retorna nil |
| `CareValidate::Ontraport::CreateCheckinData::CaseIdNotFound` | Notifica Slack canal customer support + `NotificationService`, retorna nil |

El resultado: el request queda atascado en `needs_prescriber_submission` indefinidamente, sin jobs en Dead Set ni en Retry Set.

## Señales de alerta en producción

Un request atascado en `needs_prescriber_submission` se identifica por:

1. `state = "needs_prescriber_submission"` por más tiempo del esperado (el job normal tarda segundos)
2. **`nk` y `case_nk` son `nil`** — confirma que el job nunca completó exitosamente
3. No hay jobs en `Sidekiq::DeadSet` ni `Sidekiq::RetrySet` para ese `request_id`

La ausencia de `case_nk` es el indicador clave: si el job hubiera completado, ambos campos estarían poblados.

---

## Caso real — Script 860229 (Jocelyn Smith, 2026-07-17)

### Contexto

Request ID: `019f708b-920e-7d2f-84b8-a2d35f3000b6`

El request llegó el 17 de julio de 2026 a las 14:47 UTC y quedó atascado en `needs_prescriber_submission`. A los ~5 minutos de creado ya no había avanzado.

### Diagnóstico

```
state:     needs_prescriber_submission
nk:        nil          ← job nunca completó
case_nk:   nil          ← nunca se creó/encontró un caso en CareValidate
script_nk: 860229
incoming_webhook source_event: care_validate_checkin
```

Al revisar todos los requests de la cuenta del paciente:

| Request | Estado | case_nk | Fecha |
|---|---|---|---|
| `019f3cfe...` | `canceled` | nil | 2026-07-07 |
| `019f708b...` | `needs_prescriber_submission` | nil | 2026-07-17 |

**Causa raíz**: el paciente nunca tuvo un caso creado en CareValidate. El primer request (jul 7) fue cancelado sin `case_nk`. El segundo llegó como `care_validate_checkin`, pero `SendCheckinJob` necesita un caso existente al cual hacer checkin — al no encontrarlo, lanzó `CaseIdNotFound` silenciosamente.

### Solución

Como el paciente no tenía caso previo en CareValidate, se procesó el request como un `initial_signup` usando `FindOrCreateCaseJob` directamente desde consola:

```ruby
CareValidate::FindOrCreateCaseJob.new.perform(
  "019f708b-91c2-73f2-acbe-07bfdd2d9a9c",  # incoming_webhook_id
  "019f708b-920e-7d2f-84b8-a2d35f3000b6",  # request_id
  "860229"                                   # script_id
)
```

El job creó el caso en CareValidate y avanzó el request hasta `waiting_for_prescription`:

```
state:    waiting_for_prescription
nk:       948efbc9-7190-43c3-9fa7-636863ae3716
case_nk:  3b83fc1e-006b-49ed-9b07-3e10bc8f1144
```

### Comandos de diagnóstico reutilizables

```ruby
# 1. Cargar el request
r = CareValidate::Request.find("<request_id>")

# 2. Ver estado y campos clave
puts "state: #{r.state}, nk: #{r.nk}, case_nk: #{r.case_nk}"

# 3. Verificar el webhook
w = IncomingWebhook.find(r.incoming_webhook_ids.last)
puts "source_event: #{w.source_event}, state: #{w.state}"

# 4. Ver todos los requests de la cuenta del paciente
CareValidate::Request.for_account(r.account).order(:created_at)
  .pluck(:id, :state, :case_nk, :nk, :script_nk, :created_at)

# 5. Buscar jobs fallidos (si están vacíos: el job falló silenciosamente)
Sidekiq::DeadSet.new.select { it.args.include?("<request_id>") }
Sidekiq::RetrySet.new.select { it.args.include?("<request_id>") }
```

### Cuándo aplicar la solución del caso real

Si el request cumple todo esto:
- `state = "needs_prescriber_submission"`
- `nk` y `case_nk` son `nil`
- No hay jobs en Dead/Retry para ese request_id
- El paciente no tiene ningún request previo con `case_nk` poblado

Entonces se puede usar `FindOrCreateCaseJob` directamente. El job internamente hace un `GET` a CareValidate para buscar si ya existe un caso por email — si existe, hace un checkin; si no, crea un caso nuevo. Es seguro de correr aunque no se sepa con certeza si hay caso previo en CV.
