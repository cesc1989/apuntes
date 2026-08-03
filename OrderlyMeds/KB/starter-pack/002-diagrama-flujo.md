# Diagrama de flujo — Check-in → Select Treatment → Starter Pack

Complemento visual de [`001-activacion-starter-pack.md`](001-activacion-starter-pack.md).

## 1. Flujo principal con ventana de activación

```mermaid
flowchart TD
    A["Dashboard<br/>MP: ReadyForCheckin / CheckinDelayed"] --> B["GET /checkin<br/>CheckinController#new"]
    B --> C["POST /checkin<br/>CheckinController#create"]
    C --> D{"form.valid?"}
    D -->|no| B
    D -->|sí| E["Patient::Checkin.process_form<br/>checkin.rb:35-54"]

    E --> E1["1· Assessment.record_responses"]
    E1 --> E2["2· complete_checkin!<br/>→ CheckinCompleted"]
    E2 --> E3["3· create_patient_checkin!<br/>⚠️ ÚNICO punto de creación"]
    E3 --> E4["4· CheckinDataMapper<br/>update_related_objects!"]
    E4 --> E5["5· Ajusta customer_lifecycle_stage<br/>⚠️ revierte Restarting → Existing"]
    E5 --> E6["6· mark_ready_for_product_selection!<br/>→ ReadyForProductSelection"]

    E6 --> W{{"🔧 VENTANA DE ACTIVACIÓN<br/>marcar is_starter_plan_only aquí"}}

    W --> F["GET /checkin/purchases/new<br/>PurchasesController#new"]
    F --> G["POST /checkin/purchases<br/>PurchasesController#create"]
    G --> H["create_medpicker_recommendation<br/>lee is_starter_plan_only"]
    H --> I["create_order<br/>→ Salesforce::Order"]
    I --> J{"result.needs_payment?"}
    J -->|sí| K["mark_ready_for_order_payment!<br/>→ ReadyForOrderPayment"]
    J -->|no| L["mark_order_payment_submitted!<br/>→ OrderPaymentSubmitted"]
    K --> M["Payment link · Stripe"]
    L --> N["Página de confirmación"]
    M --> N
    N --> O["Salesforce Flow out-of-band<br/>→ ReadyToCreateVisit"]

    style W fill:#fff3cd,stroke:#856404,stroke-width:3px,color:#000
    style E3 fill:#f8d7da,stroke:#721c24,color:#000
    style E5 fill:#f8d7da,stroke:#721c24,color:#000
    style H fill:#d1ecf1,stroke:#0c5460,color:#000
```

## 2. Estados del Member Period

```mermaid
stateDiagram-v2
    [*] --> ReadyForCheckin

    InHealthScreening --> HealthScreeningCompleted : mark_health_screen_completed
    HealthScreeningCompleted --> ReadyForOrderPayment : mark_ready_for_order_payment<br/>(clientes nuevos, sin check-in)

    ReadyForCheckin --> CheckinDelayed : delay_checkin_status
    CheckinDelayed --> ReadyForCheckin : mark_ready_for_checkin

    ReadyForCheckin --> CheckinCompleted : complete_checkin
    CheckinDelayed --> CheckinCompleted : complete_checkin

    CheckinCompleted --> ReadyForProductSelection : mark_ready_for_product_selection

    ReadyForProductSelection --> ReadyForOrderPayment : mark_ready_for_order_payment
    ReadyForProductSelection --> OrderPaymentSubmitted : mark_order_payment_submitted

    ReadyForOrderPayment --> ReadyForProductSelection : reset_ready_for_product_selection
    CheckinCompleted --> ReadyForCheckin : reset_ready_for_checkin
    ReadyForProductSelection --> ReadyForCheckin : reset_ready_for_checkin

    OrderPaymentSubmitted --> ReadyToCreateVisit : Salesforce Flow<br/>(out-of-band)
    ReadyToCreateVisit --> VisitCreated : create_visit

    ReadyForCheckin --> MissedCheckin : missed_checkin
    CheckinCompleted --> MissedCheckin : missed_checkin
    ReadyForProductSelection --> MissedCheckin : missed_checkin
    ReadyForOrderPayment --> MissedCheckin : missed_checkin

    note right of ReadyForProductSelection
        Estado esperado al
        activar el starter pack
    end note
```

## 3. Árbol de decisión para Customer Support

```mermaid
flowchart TD
    S["Necesito activar<br/>el starter pack"] --> Q1{"mp.patient_checkin<br/>existe?"}

    Q1 -->|no| X1["❌ El CX no completó el check-in.<br/>Pedirle que lo complete primero.<br/>NO tocar customer_lifecycle_stage:<br/>el check-in lo revertiría."]
    Q1 -->|sí| Q2{"mp.status ?"}

    Q2 -->|ReadyForProductSelection| Y1["✅ Ventana ideal.<br/>Marcar el flag.<br/>El CX selecciona tratamiento."]
    Q2 -->|"ReadyForOrderPayment /<br/>OrderPaymentSubmitted"| Y2["⚠️ Ya seleccionó tratamiento.<br/>Marcar el flag y pedirle que<br/>vuelva a /checkin/purchases.<br/>Se regenera la recomendación."]
    Q2 -->|"ReadyToCreateVisit<br/>o posterior"| X2["❌ Demasiado tarde.<br/>La visita ya está en curso.<br/>Escalar a ingeniería."]
    Q2 -->|"ReadyForCheckin<br/>(check-in cancelado)"| X3["❌ El check-in fue reseteado.<br/>El flag se perdió.<br/>Rehacer check-in y re-marcar."]

    X1 --> R["Volver a verificar<br/>tras el check-in"]
    R --> Q1

    style Y1 fill:#d4edda,stroke:#155724,color:#000
    style Y2 fill:#fff3cd,stroke:#856404,color:#000
    style X1 fill:#f8d7da,stroke:#721c24,color:#000
    style X2 fill:#f8d7da,stroke:#721c24,color:#000
    style X3 fill:#f8d7da,stroke:#721c24,color:#000
```

## 4. Efecto del flag en la MedPickerRecommendation

```mermaid
flowchart LR
    A["MedPickerRecommendationBuilder<br/>#conditional_fields"] --> B{"patient_checkin<br/>.is_starter_plan_only?"}

    B -->|true| C["return nil<br/><br/>Recomendación SIN:<br/>· prescription_strength<br/>· most_recently_prescribed_form<br/>· most_recently_prescribed_medication<br/>· most_recently_prescribed_treatment<br/>· dosage_feedback"]

    B -->|false / nil| D["Recomendación CON<br/>la dosis previa y<br/>el feedback de dosificación"]

    C --> E["✅ Starter pack:<br/>arranca desde dosis inicial"]
    D --> F["Renovación normal:<br/>continúa la titulación"]

    style C fill:#d4edda,stroke:#155724,color:#000
    style E fill:#d4edda,stroke:#155724,color:#000
```

---

Fuentes: `app/services/patient/checkin.rb`, `app/controllers/patient/checkin_controller.rb`,
`app/controllers/patient/checkin/purchases_controller.rb`,
`app/services/patient/checkin/med_picker_recommendation_builder.rb`,
`app/models/salesforce/member_period.rb`
