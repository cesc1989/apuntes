# Casa Order

## Concepto

Un **Casa Order** (`Casa::Order`) es la orden enviada a la farmacia CareValidate (CASA) para dispensar medicación. Representa el intento de ordenar un medicamento contra un caso clínico aprobado en CareValidate.

- **Modelo:** `Casa::Order` (`app/models/casa/order.rb`)
- **Tabla:** `casa_orders`
- **Relación con Order:** a través de `Casa::OrderOrder` (join table `casa_order_orders`)

## Campos importantes

| Campo | Descripción |
|---|---|
| `case_nk` | ID del caso en CareValidate (`caseId`) |
| `external_nk` | ID de la orden en CASA una vez enviada (único) |
| `status` | Estado actual de la orden |
| `payload` | JSONB con los datos enviados a la farmacia |
| `sent_payload` | JSONB con el payload exacto que se envió |
| `failure_message` | Mensaje de error si falló el envío |
| `state_history` | JSONB con auditoría de transiciones de estado |
| `sent_at` / `received_at` / `shipped_at` / `delivered_at` / `canceled_at` | Timestamps del ciclo de vida |
| `tracking_number` / `carrier` | Datos de envío |

## Estados

```
pending → submitted → received → shipped → delivered
       ↘                      ↘
        failed → pending_support_review
       ↘
        canceled
```

- Cualquier estado puede hacer `reset` → `pending` (limpia timestamps y datos de envío)
- `submit_order` solo transiciona si `case_nk` está presente

## Relación con CareValidate

Los `Casa::Order` dependen de un **caso clínico** en CareValidate identificado por `case_nk`. El caso debe estar:
- Con `status` abierto: `APPROVED`, `PENDING`, `IN_PROGRESS`, `OPEN`
- Con `closedAt` nulo

### Bug conocido (CareValidate)

CareValidate no limpia `closedAt` al reabrir un caso. El campo puede tener una fecha aunque el `status` sea `APPROVED`. Nuestro código en `Casa::ClassifyOrderFailure` (`app/services/casa/classify_order_failure.rb`) verifica:

```ruby
return :case_closed if detail["closedAt"].present?
```

Esto causa falsos positivos cuando un caso fue cerrado y luego reabierto. La solución es pedirle a CareValidate que nuleen `closedAt` en su backend al reabrir.

## Clasificación de fallos (`Casa::ClassifyOrderFailure`)

Cuando una orden falla, este servicio consulta el caso en CareValidate y devuelve una de estas razones:

| Razón | Descripción |
|---|---|
| `:no_case` | `case_nk` vacío o caso no encontrado en la API |
| `:case_closed` | `closedAt` presente en la respuesta de CareValidate |
| `:no_approved_decision` | No hay decisiones con `isApproved: true` y `esigned: true` |
| `:unknown` | Error inesperado o condición no contemplada |

Las razones `PERMANENT_REASONS = [:no_case, :case_closed, :no_approved_decision]` no se reintentan — van a `pending_support_review`.

## Scopes útiles

```ruby
Casa::Order.pending
Casa::Order.submitted
Casa::Order.failed
Casa::Order.pending_support_review
Casa::Order.current  # pending + submitted + received + shipped + delivered + pending_support_review
```

## Script de diagnóstico

Para detectar órdenes activas cuyo caso tiene `closedAt` presente pero `status` abierto (bug de CareValidate):

```ruby
OPEN_STATUSES = %w[APPROVED PENDING IN_PROGRESS OPEN].freeze

orders = Casa::Order
  .where.not(case_nk: nil)
  .where(status: %w[pending submitted failed pending_support_review])
  .order("RANDOM()").limit(5)

orders.each do |casa_order|
  detail = CareValidate::Api::Case.fetch_by_case_id(case_id: casa_order.case_nk)
  next if detail.blank?

  if detail["closedAt"].present? && OPEN_STATUSES.include?(detail["status"])
    puts "PROBLEMA: casa_order=#{casa_order.id} case=#{casa_order.case_nk} status=#{detail["status"]} closedAt=#{detail["closedAt"]}"
  end
rescue => e
  puts "ERROR: casa_order=#{casa_order.id} — #{e.message}"
end
```

## Archivos clave

| Archivo | Propósito |
|---|---|
| `app/models/casa/order.rb` | Modelo principal con state machine |
| `app/services/casa/classify_order_failure.rb` | Clasifica el motivo de fallo de una orden |
| `lib/care_validate/api/case.rb` | Cliente API para casos de CareValidate (solo lectura/creación, sin update) |
