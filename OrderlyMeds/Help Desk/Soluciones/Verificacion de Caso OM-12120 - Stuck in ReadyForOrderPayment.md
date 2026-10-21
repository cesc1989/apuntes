# Cómo leer la salida de la verificación en el caso OM-12120 - Stuck in ReadyForOrderPayment

Claudio me dio una verborrea. Entonces para no hacer el texto de la descripción del problema pongo acá cómo leer la salida cuando se está verificando.

Lee las salidas en orden. Cada paso decide si sigues o te detienes.

## 1. `mp.slice(...)`: ¿en qué estado está el período?

- **`status`:** debe ser `ReadyForOrderPayment`. Si ya es `ReadyToCreateVisit` o un estado posterior, el Flow corrió y solo faltaba la sincronización: ==🟡no toques nada.==
- **`checkin_deadline_date`:** si ya pasó, hay urgencia. `CancelMissedMemberPeriodsJob` puede pasarlo a `MissedCheckin` y dejar los loyalty points en Zero.

## 2. `mp.orders.map`: ¿cuántas órdenes y links hay?

- **Una orden con link `Active`:** el pago no llegó o se está procesando. No es este patrón: ve a los casos A, B o C de la tabla del paso 4.
- **Dos o más órdenes:** señal del patrón duplicado. Normalmente una tiene el link `Disabled` (la pagada) y otra `Active` (la duplicada).
- Anota los `omid`.

## 3. `acct.payments`: ¿se cobró realmente?

- Busca un pago **`Processed`** por el **monto** del link y con la fecha del cobro.
- **Sin pago:** no la empujes. O el cobro no se hizo, o falta el `Payment` (caso C).
- Copia el **`paymentintentguid`** para el paso 4.
- **Línea de tiempo:** si el pago es **anterior** a la orden más reciente, se pagó una orden vieja y luego se creó otra.

Para sacar la fecha de un omid (UUIDv7):
```ruby
Time.at("01a13ccb-924a".delete("-").to_i(16) / 1000.0).utc
```

## 4. PaymentIntent + PNP + PaymentLink: ¿dónde quedó el procesamiento?

**PaymentIntent**
- `Status: Succeeded` y `IsCaptureComplete: true`: el dinero se capturó.
- `PaymentLinkId`: dice **con qué link pagó**. Compáralo con los del paso 2.

**PNP (PaymentNeedingProcessing)**

| Resultado | Significado | Qué hacer |
|---|---|---|
| `ProcessingComplete` | Salesforce terminó de procesar | Revisa el link y las órdenes (duplicado o Flow pisado) |
| `NeedsProcessing` / `Failed` | Se quedó a medias (caso B o C) | Reintentar el PNP, no forzar el estado |
| No existe | No se creó el PNP (caso A) | Crearlo, como hace `MissingPnpReconciliationJob` |

**PaymentLink del PI**
- Debe estar `Disabled`.
- `orderid__r__omid__c` dice **cuál orden fue la pagada**.
- Si **no** es la orden más reciente del paso 2, se confirma el patrón duplicado.

## 5. Órdenes y recomendaciones: ¿se puede usar la más reciente?

- **Órdenes:** `product2__omid__c`, `quantity` y `unitprice` deben ser idénticos.
- **Recomendaciones:** compara `productid__r__omid__c`, `recommendedmedicationid__r__omid__c`, `recommendedquantity__c` y `recommendedrxstrength__c`.
  - **Coinciden:** cancela la **vieja**. El scheduler usa `latest_med_picker_recommendation`.
  - **No coinciden:** cancela la **nueva**, para que la última activa sea la pagada.
  - **La pagada ya está `Canceled`:** detente. Hay que reactivarla y es otro procedimiento.
- Al final debe quedar **una sola** en `RecommendationMade`.

## 6. `order_summaries`: ¿la farmacia va a encontrar la orden?

- Debe existir un summary creado segundos después del pago, con `originalorderid` igual al `sfid` de la **orden pagada**.
- ==🟡**Si no existe:** detente. Cuando la farmacia confirme, `create_fulfillment_order` va a fallar con "Cannot find order to fulfill".==

## 7. `clinical_encounters`: ¿hay algo que choque?

- **`[]`:** puedes seguir.
- **`Pending` o `Scheduled`:** detente. Ya hay una visita en curso o el scheduler va a rechazar una nueva.

## Checklist para aplicar la corrección

- [ ] MP en `ReadyForOrderPayment`
- [ ] Payment `Processed` por el monto correcto
- [ ] PI `Succeeded` y PNP `ProcessingComplete`
- [ ] El link pagado está `Disabled` y pertenece a una orden **anterior**
- [ ] Mismo producto, dosis y precio en las órdenes y las recomendaciones
- [ ] Existe un OrderSummary de la orden pagada
- [ ] No hay clinical encounters

Si falla cualquier punto, no es este patrón y no conviene forzar `ReadyToCreateVisit`.