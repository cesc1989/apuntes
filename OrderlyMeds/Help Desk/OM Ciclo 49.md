# Ciclo 49 de OrderlyMeds Support

> [!Info]
> Del Jueves 18 de Junio al Miércoles 1 de Julio.

## Caso OM-9481 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("paigekaye@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9350 - Reset Check In CX en Prospect 🟢

Etiquetas: #om_cx_in_prospect

Dicen esto:
> Customer wants to order, her acct was merged previously. Please reset link so cx can order thank you

Pero cuando voy al perfil del cliente en Success sale que estado es "Prospect"

### ¿Qué significa Prospect?

Según Jay:
> means that cx haven't placed an order yet

### CX debe completar el HealthScreening

Para resolver el cliente debe completar esta parte (hacer una orden). Se debe ubicar el enlace que lleva a esta parte en Ontraport. Está en la sección "Activity" del perfil del CX. Buscar el campo "ShortUrl Healthscreening".

Copiar e indicar a CS que deben pedirle al CX que complete esa etapa primero.

## Caso OM-9473 - Oops Error 🟢

Etiquetas: #om_oops_error 

Todos:

- [x] Revisar el account
- [x] Actualizar el campo `workos_user_nk`
- [x] Sincronizar salesforce

## Caso OM-9426 - Stuck in Submitted 🟢

Seguí los pasos del caso [[OM Ciclo 48#Caso OM-9337 - Stuck in submitted 🚨]]:

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```

## Caso OM-9489 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("amandafalco11@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9400 - Remove CC de Salesforce 🟢

Etiquetas: #om_remove_credit_card

Pasos:

- Busca el `Account` del CX
- Luego lee las CCs que tenga adjuntas con `account.salesforce_account.card_payment_methods`
- Si solo es una, o a la que se necesite, la actualizas el campo `status` a `"Inactive"`

```ruby
account = Account.find("CHANGEME")

sfacc = account.salesforce_account
ccs = sfacc.card_payment_methods

ccs.first.update!(status: "Inactive")
```

## Caso OM-9347 - Script Error 🟢

Etiquetas: #om_script_error 

Cuando se revisa el Script en Ontraport el Outcome dice "Script Error". A pesar de hacer un resubmit sigue dando el mismo problema.

### Sin solución? Cancel & Refund ℹ️

> [!Info]
> Esto como que fue una situación dada por el paso al plan de 2 meses. CS tiene que hacer cancel & refund.

Este sería el caso para medicamentos de 3 meses. Fueron deshabilitados.

Mensaje para CS:
```
Currently, the 3-month option is no longer available. Therefore, a cancellation and full refund are required. The customer will then need to complete a new checkin and select either the 1-month or 2-month option.
```

## Caso OM-9396 - Invalid Drug ID - SF 🟢

Etiquetas: #om_invalid_drug_id

> [!Note]
> Este es lo mismo que el caso OM-9347. La inhabilitación del plan de 3 meses hace que este MP quede inválido. Se necesita hacer cancel & refund y nuevo MP.

Le pregunté a Fabian y me dijo que es lo mismo que el OM-9347. Dejé el mensaje en el hilo y moví el caso a "waiting on support".

### Verificación que está en la opción de tres meses

Para esto fui a Salesforce y revisé el Member Period. Pude ver en la sección "Med Picker Recommendation" que dice en "Product" "3-month Tirzepatide (High Dosage)". Ahí queda claro lo que me dijo Fabian.

## Caso OM-9426 - Stuck in Submitted 🟢

Seguí los pasos del caso [[OM Ciclo 48#Caso OM-9337 - Stuck in submitted 🚨]]:

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```

## Caso OM-9573 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("nikki199043@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9577 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("mbernar9@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9398 - Stuck in Active 🟢🔑

Etiquetas: #om_stuck_in_active

> [!Important]
> La clave de esto es verificar el log del webhook en Ontraport. Revisar el flujo en Case Overview para detectar cualquier cosa que pueda impedir que no se complete.

Para este caso lo que hizo Fabian fue verificar que al hacer el resubmit hubiera algún registro en la lista de logs. Para verlos se va a [esta página](https://app.ontraport.com/#!/webhook_log/listAll) y se busca por el ID del Script.

> [!Info]
> Sobre los webhooks de Ontraport:
>
> - Siempre se crea un webhook. Si hay fallo o éxito, eso no le importa a Ontraport.
> - Los logs duran 24 horas
>
> Este caso tuvo  la particularidad que nunca generó el webhook. Eso fue el indicio para hacer lo a continuación.

### Verificación de Dosage en Case Overview  🔎

Se empezó revisando el detalle del submission en Case Overview. La pista que siguió Fabian fue que notó un cambio en el campo Dosage.

Inicialmente era ==Semaglutide 1.5mg y en un punto cambió a necesitar ser .25mg==. En Ontraport nunca se reflejó ese cambió lo que llevó a la sospecha que eso necesitaba cambiar.

> [!Important]
> Esto también se puede comprobar en la sección "Requested MedPicker Data" en el detalle del Care Validate Request. Para este caso ahí sale que esta es la dosis deseada:
> ```ruby
> "rxStrength": 0.25,
> "opFieldDosage": "Semaglutide 0.25mg",
> "productStrength": 0.25,
> ```

Para corregir entonces hizo los siguientes cambios en el `IncomingWebhook` del `CareValidate::Request`.

### Paso 1: Actualización manual del payload del IncomingWebhook

Lo primero que hizo fue ubicar el Request e inspeccionar que tuviera valor en `incoming_webhook_ids`:
```ruby
request = CareValidate::Request.find("019e9877-d0fd-7660-ad53-2793fd8301a3")
```

Después, se inspecciona dicho incoming webhook:
```ruby
existing_webhook = IncomingWebhook.find(request.incoming_webhook_ids.first)
existing_webhook.data
```

> [!Important]
> `data` es una función de `IncomingWebhook` que lee lo que se guardó en `payload`. Por su parte `payload` es un objeto de ActiveStorage.

Esto es un ejemplo de `payload`:
```ruby 
{
  "contactId" => "427226",
  "scriptId" => "785154",
  "firstName" => "Lalo",
  "lastName" => "Landa",
  "dob" => "",
  "phone" => "",
  "email" => "",

  "visitType" => "weightlossfollowup",
  "2monthsstarter" => "No",

  "patientPreference" => [
    {
      "name" => "OM Core: Semaglutide - $199/month - 4 Injections",
      "numberMonths" => "1",
      "strength" => "Semaglutide .25mg",
      "change" => "Discuss Decreasing Dosage",
      "ingredients" => "B3,B5",
      "titrateup" => "I want multiple months of the same dose",
      "medIdOverride" => "",
      "refills" => "0"
    }
  ],

  "Q1" => "What is your Height?",
  "A1" => "5'4",
}
```

Lo que había que actualizar es la llave `strength` dentro de `patientPreference`. Para eso había que copiar el contenido totalmente y luego reasignar:
```ruby
updated_data = existing_webhook.data.deep_dup
updated_data["patientPreference"][0]["strength"] = "Semaglutide .25mg"
```

Ahora si procedemos a actualizar todo.

#### Limpiar caché del método `IncomingWebhook#data`

Hay que hacer esto para invalidar la caché ya que esa función hace memoization del valor.
```ruby
existing_webhook.instance_variable_set(:@data, nil)
```

#### Actualizar payload de `IncomingWebhook`

Y ahora reemplazamos el payload anterior con el nuevo corregido:
```ruby
existing_webhook.payload.attach(                                
  io: StringIO.new(updated_data.to_json),
  filename: "payload.json",
  content_type: "text/json"
)
```

> [!Note]
> Aquí se hace lo mismo que en `IncomingWebhook.create_from_payload`. En vez de crear un nuevo incoming webhook solo se crea un nuevo blob y se relaciona.

Esto crea un nuevo blob y reemplaza el anterior.

### Paso 2: Botón Resubmit Latest Ontraport Webhook

Está en la sección "Incoming Webhooks" en el detalle del CareValidate Request. Después de hacer esto hay que ir a esta sección y clicar este botón.

### Comprobación Final

Al revisar de nuevo el CareValidate Request en Success se ve que está en estado "Waiting For Prescription" y el Script en Ontraport está en "Pharmacy Selected".

## Caso OM-9564 - Stuck in Submitted 🟢

```ruby
request = CareValidate::Request.find("")
request.update!(state: "needs_resubmission")
```

No tuve que hacer nada porque ya la orden fue enviada.

## Caso OM-9562 - Stuck in Submitted 🟢

Nada que hacer. Ya estaba el Script en "Order at Pharmacy".

## Caso OM-9419 - Oops Error 🟢

Etiquetas: #om_oops_error 

- [x] Revisar el account

```ruby
account = Account.find_by(email: "")
account.workos_user_nk
```

- [x] Actualizar el campo `workos_user_nk`
```ruby
account.update!(workos_user_nk: "")
```

- [x] Sincronizar salesforce
```ruby
Salesforce::CustomerUser.create(
  salesforce_person_account: account.salesforce_account,
  local_account: account
)
```

## Caso OM-9483 - Beluga Missing Values 🔵

Etiquetas: #om_beluga  #om_script_error 

Lo tenía pero lo pusieron urgente y Fili reasignó a Fabian. Lo primero que preguntó Fabian fue:
> Can you please confirm whether the customer has any sensitivities? That seems to be what's causing the error to appear.

Y mostró captura de la parte en Ontraport donde están las sensitivities.

> [!Info]
> Las sensitivities están en el perfil en Ontraport. Hay que ir a la página "Health Info" que está en el menú de la izquierda.

El Outcome del Script es "Script Error".

> [!Note]
> Lo mismo que pasó con [[OM Ciclo 48#Caso OM-9215 - Script error - Beluga 🟢ℹ️]]


## Caso OM-9582 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("gaela87@yahoo.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9596 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("tpatel021@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9599 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("njlograsso@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```


## Caso OM-9590 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("skhansen1962@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9598 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("ramimperry@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9600 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("trenton.holland@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9604 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("amandawilson1027@icloud.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9605 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("swcampbell33@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9569 - Prescribed but error in CV channel 🟡ℹ️

Etiquetas: #om_prescribed #om_care_validate_error

Dice:
> Already prescribed but is showing an error in CV channel.

En la captura que pasan dice:
> We encountered an internal error. Please try again.

![[om_9569.01.png]]

Jaime me dijo que toca, desde Salesforce, que hagan un "Resubmit to MSO":
![[om_9569.02.png]]

> [!Note]
> Hay que mirar bien cuál es el Member Period a hacer resubmit. Luego eso abre un formulario y hay que llenarlo bien. Mejor que lo haga CS.

Jaime me dice que hay que hacer esto para que el sistema vuelva a hacer sync de la medicina con información nueva.

## Caso OM-9593 - Stuck in Submitted 🟢

```ruby
request = CareValidate::Request.find("")
request.update!(state: "needs_resubmission")
```

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```

## Caso OM-9594 - Stuck in Submitted 🟢

> [!Note]
> El estado del request de este caso es `needs_prescriber_submission`.

```ruby
request = CareValidate::Request.find("")
request.update!(state: "needs_resubmission")
```

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```

## Caso OM-9398 - Fallo al crear el order 🟡🔑

> [!Important]
> Esto es una continuación de [[OM Ciclo 49#Caso OM-9398 - Stuck in Active 🟢🔑]]. Lo descrito en el anterior bloque sí sirvió para desbloquear pero este Script tiene otro problema. Es el que se intentará resolver a continuación.

### ¿Por qué se reabre?

Comentaron esto:
> I know this has been just resubmitted. However, outcome is prescribed. Bellow is the error

Con esta captura de Ontraport:
![[om_9398_03.png]]

Pregunté a Fabian y me dijo:
> significa que ==esta prescrito pero que fallo a la hora de crear al orden==. Es un problema aparte.

### Pistas

Revisando en Care Overview, en la pestaña "Casa Pharmacy Orders" aparece esto:
![[om_9398.04.png]]

Nota el label amarillo que dice "pending_support_review". Esa es la pista que me dio Fabian.

Esto denota un error del sync en `Casa::SubmitCasaOrder`. Snippet:
```ruby
if Casa::ClassifyOrderFailure.permanently_unorderable?(failure_reason)
	casa_order.mark_pending_support_review!
end
```

##### ¿Por qué Casa y no PerfectRx?

Porque cuando se revisa la pestaña "Orders" en Care Overview, para este caso, se ve que está pendiente la de Casa.

![[om_9398.05.png]]

### Revisando la Orden en la BD

> [!Note]
> El ID de `Casa::Order` está en el registro que aparece en la pestaña "Casa Orders" en Care Overview. Está en fondo gris.

Fabian indica revisar esto en la base de datos usando parte del código de la clase `Casa::SubmitCasaOrder`. Precisamente:
```ruby
casa_order = Casa::Order.find("019ef238-5739-74ce-88df-7ea6526aef27")
failure_reason = Casa::ClassifyOrderFailure.call(casa_order: casa_order, logger: Logger.new($stdout))
```

Eso da una razón del fallo. En este caso fue: `:case_closed`.

> [!Info]
> Si un case está cerrado la API no los acepta y rechaza la petición.

### Pedir reabrir el case

> [!Note]
> El ID del case está en la pestaña "Care Validate Submissions". El la columna "Case NK".

Hay que ir al canal `#om-carevalidate-patient-exp` y pedir que reabran el caso con este formato:
```
:wave: @Princess Joyce Ruth Chua @Marissa @Jaylian Omambac
Name: Monique Davis
Email: monique213davis@gmail.com
Case link: https://accommodations.carevalidate.com/accommodations/cases/fd46e3be-7d7c-4b9e-a753-6bb6220c8c20
Concern/Request: Please reopen case `fd46e3be-7d7c-4b9e-a753-6bb6220c8c20`. It needs to be resubmitted.
```

### Comprobar case está abierto

Con este código se comprueba el estado:
```ruby
casa_order = Casa::Order.find "019ef238-5739-74ce-88df-7ea6526aef27"
failure_reason = Casa::ClassifyOrderFailure.call(casa_order: casa_order, logger: Logger.new($stdout))
```

Debería devolver algún estado que no sea:
```ruby
:no_case
:case_closed
:no_approved_decision
```

Según como indica la clase `Casa::ClassifyOrderFailure`.

### Reenviar el order

Para que se pueda continuar el proceso:
```ruby
casa_order = Casa::Order.find "019ef238-5739-74ce-88df-7ea6526aef27"
casa_order.reset!
Casa::SubmitCasaOrder.call(casa_order: casa_order, logger: Rails.logger)
```

## Caso OM-9621 - Stuck in Submitted 🟢ℹ️

```ruby
request = CareValidate::Request.find("")
request.update!(state: "needs_resubmission")
```

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```

### Sobre Script ya en Pharmacy Selected

El caso decía que estaba "Stuck in Submitted" y me fue asignado. Cuando fui a ver el Script ya estaba en "Pharmacy Selected". Lucía así:
![[om_9612.01.png]]

En todo caso hice el resubmit y luego salió lo que esperaba ver del webhook:
![[om_9621.02.png]]

## Caso OM-9608 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("nicolemjk22@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9613 - Oops Error 🟢

Etiquetas: #om_oops_error 

- [x] Revisar el account

```ruby
account = Account.find_by(email: "")
account.workos_user_nk
```

- [x] Actualizar el campo `workos_user_nk`
```ruby
account.update!(workos_user_nk: "")
```

- [x] Sincronizar salesforce
```ruby
Salesforce::CustomerUser.create(
  salesforce_person_account: account.salesforce_account,
  local_account: account
)
```