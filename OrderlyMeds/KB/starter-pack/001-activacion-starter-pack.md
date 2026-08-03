# Activación del Starter Pack

## TL;DR para Customer Support

> **El CX debe completar el check-in ANTES de que podamos activar el starter pack.**
>
> El registro de "Patient Check In" en Salesforce **no existe** hasta que el CX termina el check-in. Sin ese registro no hay dónde marcar el flag.
>
> Y si se marca `Restarting` en el Member Period *antes* del check-in, **el propio check-in lo revierte** a `Existing`. Trabajo perdido.

**Ventana correcta de activación:** después de que el CX envía el check-in y **antes** de que envíe la selección de tratamiento.

---

## El flag

`Salesforce::PatientCheckin#is_starter_plan_only` → alias de `isstarterplanonly__c`
(`app/models/salesforce/patient_checkin.rb:45`)

Ningún código de Rails escribe este campo. `CheckinDataMapper` no lo toca — el único lugar donde aparece en el modelo es el `alias_attribute`. Se setea desde Salesforce o manualmente vía consola.

---

## Flujo: Check-in → Select Treatment

### Paso 1 — Check-in

| | |
|---|---|
| Ruta | `GET /checkin` → `POST /checkin` |
| Controller | `Patient::CheckinController#new` / `#create` |
| Servicio | `Patient::Checkin.process_form` (`app/services/patient/checkin.rb:35-54`) |

`process_form` hace, en una sola transacción:

1. Registra las respuestas → `Salesforce::Assessment.record_responses` (`checkin.rb:38-43`)
2. `member_period.complete_checkin!` → `ReadyForCheckin`/`CheckinDelayed` → **`CheckinCompleted`**
3. **`member_period.create_patient_checkin!(...)`** ← ***único lugar en toda la app donde se crea el `PatientCheckin`*** (`checkin.rb:45`)
4. `CheckinDataMapper#update_related_objects!` (`checkin.rb:46`)
5. Ajusta `customer_lifecycle_stage` (ver [Trampa #1](#trampa-1--restarting-puesto-antes-del-check-in-se-revierte))
6. `member_period.mark_ready_for_product_selection!` → **`ReadyForProductSelection`**

Luego el controller redirige a `new_checkin_purchases_path` (`checkin_controller.rb:112-114`).

### Paso 2 — Select Treatment

No existe un controller llamado "treatment". Es `Patient::Checkin::PurchasesController`.

| | |
|---|---|
| Ruta | `GET /checkin/purchases/new` → `POST /checkin/purchases` |
| Controller | `app/controllers/patient/checkin/purchases_controller.rb` |
| Form | `Patient::Checkin::PurchaseForm` (+ `TreatmentPicker`, `ProductPicker`, `AddOnPicker`) |
| Label en UI | `t("patient.dashboards.statuses.actions.select_treatment")` |

En el `POST` (`purchases_controller.rb:35-67`), dentro de una transacción:

1. `create_medpicker_recommendation` → escribe `Salesforce::MedPickerRecommendation`
2. `create_order` → `Patient::Purchasing.create_order` → `Salesforce::Order`
3. Bifurca según si requiere pago:
   - **Requiere pago** → `mark_ready_for_order_payment!` → `ReadyForOrderPayment` → redirige al payment link (Stripe)
   - **No requiere pago** → `mark_order_payment_submitted!` → `OrderPaymentSubmitted` → redirige a confirmación

### Paso 3 — Pago

Fuera de la app (Stripe payment link) o confirmación directa. De ahí, Salesforce Flow mueve el MP a `ReadyToCreateVisit` **out-of-band** (ver comentario en `member_period.rb:13-14`).

---

## Estados del Member Period

`Salesforce::MemberPeriod` usa la gema `state_machine` sobre `status` (`app/models/salesforce/member_period.rb:3-120`). Es la única fuente de verdad de "dónde está el paciente en el funnel".

| Evento | Transición | Disparado por |
|---|---|---|
| `complete_checkin` | `ReadyForCheckin`, `CheckinDelayed` → `CheckinCompleted` | `checkin.rb:44` |
| `mark_ready_for_product_selection` | `CheckinCompleted` → `ReadyForProductSelection` | `checkin.rb:52` |
| `mark_ready_for_order_payment` | `ReadyForProductSelection`, `HealthScreeningCompleted` → `ReadyForOrderPayment` | `purchases_controller.rb:52` |
| `mark_order_payment_submitted` | `ReadyForProductSelection` → `OrderPaymentSubmitted` | `purchases_controller.rb:55` |
| `reset_ready_for_checkin` | `CheckinCompleted`, `ReadyForProductSelection` → `ReadyForCheckin` | `checkin_controller.rb:109` |
| `reset_ready_for_product_selection` | `ReadyForOrderPayment` → `ReadyForProductSelection` | `checkin_controller.rb:108`, `purchases_controller.rb:83` |

**Nota:** `HealthScreeningCompleted` puede ir directo a `ReadyForOrderPayment` (`member_period.rb:55`). El check-in es solo para ciclos de renovación, **no** para la primera orden de un cliente nuevo — ese pasa por el health screening hosteado en Salesforce.

---

## Dónde se lee el flag

El flag se consume **en el paso 2**, cuando el CX envía la selección de tratamiento. Por eso hay que marcarlo antes.

### 1. `PurchaseForm#starter_plan?`
`app/forms/patient/checkin/purchase_form.rb:61-63` → usado por la vista `app/views/patient/checkin/purchases/new.html.erb:20`

### 2. `MedPickerRecommendationBuilder#conditional_fields`
`app/services/patient/checkin/med_picker_recommendation_builder.rb:40-50`

```ruby
def conditional_fields
  return if member_period.patient_checkin.is_starter_plan_only

  {
    prescription_strength: previous_medication.titration_final_rx_strength,
    most_recently_prescribed_form: previous_medication.product.form,
    most_recently_prescribed_medication: previous_medication,
    most_recently_prescribed_treatment: previous_medication.product.treatment,
    dosage_feedback: member_period.patient_checkin.dosage_feedback
  }
end
```

Si es starter plan, la `MedPickerRecommendation` se crea **sin** dosis previa ni feedback de dosificación — que es exactamente el punto del starter pack. Si el flag se marca *después* del submit, la recomendación ya salió con la dosis previa.

### 3. Productos starter
`ProductPicker#product_options` etiqueta con `" (starter)"` los productos donde `product.is_starter_pack` (`product_picker.rb:45-46`). Ese es un campo distinto, del producto (`Salesforce::Commerce::Product#is_starter_pack` → `isstarterpack__c`), no del check-in.

---

## Código de activación

```ruby
mp = Salesforce::MemberPeriod.find_by(name: "MP-00569524")

# Falla con NoMethodError si es nil → el CX no completó el check-in.
check_in = mp.patient_checkins.last
check_in.update!(is_starter_plan_only: true)

mp.update!(customer_lifecycle_stage: "Restarting")
```

El orden importa: el `update!` del check-in es el que hace el trabajo real. El `customer_lifecycle_stage` es consecuencia — de hecho `process_form` lo derivaría automáticamente del flag si el flag ya estuviera puesto al momento del check-in (`checkin.rb:47-48`).

Verificación previa recomendada:

```ruby
mp = Salesforce::MemberPeriod.find_by(name: "MP-00569524")
mp.status                    # esperado: "ReadyForProductSelection"
mp.patient_checkin.present?  # debe ser true
```

---

## Trampas

### Trampa #1 — `Restarting` puesto antes del check-in se revierte

`app/services/patient/checkin.rb:47-51`:

```ruby
if patient_checkin.is_starter_plan_only
  member_period.update!(customer_lifecycle_stage: "Restarting")
elsif member_period.customer_lifecycle_stage == "Restarting"
  member_period.update!(customer_lifecycle_stage: "Existing")
end
```

El `patient_checkin` recién creado nace sin `is_starter_plan_only` (nada en Rails lo setea al crearlo). Entonces cae en el `elsif` y **pisa `Restarting` → `Existing`**.

### Trampa #2 — Si el CX rehace el check-in, el flag se pierde

`perform_reset_ready_for_checkin` (`app/models/salesforce/member_period.rb:122-128`) hace `patient_checkin&.cancel!` y luego `process_form` crea uno **nuevo**, sin heredar el flag. Hay que volver a marcarlo.

El `PatientCheckin` viejo queda con `status: "Canceled"`, accesible vía `mp.canceled_patient_checkins` (`member_period.rb:242`). La asociación `mp.patient_checkin` excluye los cancelados (`member_period.rb:241`).

### Trampa #3 — Si ya seleccionó tratamiento, sí se puede corregir

No hace falta rehacer el check-in. Al volver a `/checkin/purchases`, `reset_order_if_updated` (`purchases_controller.rb:81-86`) devuelve el MP a `ReadyForProductSelection`, desactiva el payment link y regenera la `MedPickerRecommendation` en el nuevo submit.

Procedimiento: marcar el flag → pedirle al CX que vuelva a seleccionar tratamiento.

---

## Referencias

| Archivo | Rol |
|---|---|
| `app/services/patient/checkin.rb:35-54` | `process_form` — crea el `PatientCheckin`, dispara transiciones |
| `app/controllers/patient/checkin_controller.rb:103-118` | Valida, resetea estados, redirige a purchases |
| `app/controllers/patient/checkin/purchases_controller.rb:35-67` | Select treatment — crea recomendación y orden |
| `app/forms/patient/checkin/purchase_form.rb:61-63` | `starter_plan?` |
| `app/services/patient/checkin/med_picker_recommendation_builder.rb:40-50` | Omite dosis previa si es starter plan |
| `app/models/salesforce/member_period.rb:3-134` | State machine + resets |
| `app/models/salesforce/patient_checkin.rb:45` | `alias_attribute :is_starter_plan_only` |
| `config/routes/patient.rb:36-46` | Rutas de checkin y purchases |

Ver también: [`001-member-period.md`](001-member-period.md)
