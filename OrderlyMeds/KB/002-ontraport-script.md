# Ontraport Script (Prescripción)

## Concepto

Un **Script** en Ontraport es un **registro de prescripción médica** — representa la consulta y receta de un paciente para un protocolo de tratamiento (principalmente medicamentos para pérdida de peso como GLP-1/Semaglutide).

Es un objeto personalizado en el CRM Ontraport con object ID `10000`.

- **Modelo Rails:** `Ontraport::Script` (`app/models/ontraport/script.rb`)
- **Tabla:** `ontraport_scripts` (almacena payload JSONB encriptado)
- **API:** `lib/ontraport/api/script.rb`
- **Meta layer:** `lib/ontraport/meta/script.rb`

## Qué contiene un Script

| Categoría | Campos |
|---|---|
| Visita | Fecha de visita, tipo (`weightloss`, `weightlossfollowup`) |
| Prescripción | Medicamento, dosis actual, texto de prescripción |
| Paciente/Médico | `contact_id` del paciente, `master_id` (ID único de la receta) |
| Farmacia | Farmacia seleccionada, tracking |
| Facturación | Invoice ID, monto, fecha |
| Seguimiento | Fecha próxima consulta, enlace a video visita |
| Dosificación | Soporte para protocolos multi-mes, tracking de aumento de dosis |

El modelo define 50+ campos mapeados a códigos de Ontraport (`f2124`, `f2125`, etc.).

## Estados posibles (outcomes)

```
prescribed → active → check_in → shipped → delivered
                    ↘ cancelled
                    ↘ order_at_pharmacy
```

13 estados en total definidos como enum en `lib/ontraport/api/script.rb`.

## Relaciones

- **Script → Order:** Un script puede tener una orden asociada (tabla join `order_ontraport_scripts`)
- **Script → Contact:** Cada script pertenece a un paciente via `contact_id`
- **Script → Script previo:** Los scripts de follow-up referencian al script anterior

## Flujo de negocio

1. El prescriptor (Beluga Health o Care Validate) escribe una receta → se crea un Script en Ontraport
2. Ontraport dispara un webhook `create_or_update_script` → se sincroniza al Rails DB (`app/controllers/webhooks/ontraport_controller.rb`)
3. El sistema crea/actualiza una `Order` basada en el Script
4. El outcome del Script avanza conforme avanza el ciclo (`prescribed` → `shipped` → `delivered`)
5. Para visitas de seguimiento, `CreateNextCheckinScriptJob` crea el próximo Script

## Archivos clave

| Archivo | Propósito |
|---|---|
| `app/models/ontraport/script.rb` | Modelo Rails con datos encriptados |
| `lib/ontraport/api/script.rb` | Mapeo de campos y CRUD hacia la API de Ontraport |
| `lib/ontraport/meta/script.rb` | Queries de alto nivel (`get_by_contact_id`, `get_by_id`) |
| `app/services/orders/find_script.rb` | Localiza scripts por `master_id` o `care_validate_request_id` |
| `app/sidekiq/care_validate/create_next_checkin_script_job.rb` | Crea el siguiente script de check-in |
| `app/controllers/webhooks/ontraport_controller.rb` | Recibe webhooks de Ontraport |

## Contexto de migración

Ontraport es el sistema **legado**. La plataforma está siendo migrada activamente a Salesforce. Los Scripts son el punto de partida para crear órdenes en el sistema antiguo.
