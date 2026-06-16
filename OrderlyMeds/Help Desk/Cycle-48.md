# Ciclo 48

## Caso OM-9224 - Resend Invoice 🟢

Etiquetas: #om_resend_invoice

Dicen esto:
> Customer request an invoice copy that clearly says 'invoice copy'. Details should be like this: (she's kinda irate)
> • Name of person who received the item or service  
> • Description of the item or service  
> • Date of service  
> • Retailer or service provider name  
> • Amount you paid

Al ver el perfil en OM se ve que es un cliente en Salesforce. Por esto, Jaime me explicó que hay un botón para reenviar un invoice al presionar "Create New Case" y elegir el Case Type.

### Sobre la fecha del Invoice

En el ticket, Ronalie comentó:
> please send the 4/27 invoice

Cuando voy a ver los MPs en Salesforce no veo ninguna fecha que corresponda a la indicada.
![[om_9224.01.png]]

Al preguntar a Jaime me explica que es la fecha del Order. Se ve en el detalle del Member Period en Salesforce en la sección "Order Summaries".
![[om_9224.02.png]]

### Dudas sobre el Invoice reenviado

Lo que pide el cliente es muy específico y no me queda claro si el mailer que indicó Jaime en el repo correspondo a esa acción que efectúa Salesforce.

El mailer que dice Jaime está en la ruta `app/views/patient/order_summary_mailer/send_invoice.html.erb`.

No me suena porque el mailer tiene son estos datos:

- first_name
- last_name
- @order.order_reference_number
- @order.effective_date

Esto:
```ruby
<% @order.order_items.each do |item| %>
	<li>Product & Plan: <%= item.product.name %></li>
	<li>Preferred Treatment: <%= item.product.humanize[:treatment] %></li>
<% end %>
```

- Subtotales
- Shipping Address

No tiene nada de lo que piden en particular.

## Caso OM-9214 - Bundle Issue 🔵

Etiquetas: #om_bundle_issue

> [!Info]
> La clave de este caso es que Beluga prescribió una medicina que no era la correcta o faltó agregar una recomendación.
> 
> (ver recomendaciones del MP en Salesforce en la sección "Med Picker Recommendation").

> [!Warning]
> Estos casos (hasta nuevo aviso) hay que escalar a Amber de Beluga para preguntar sobre dicho bundle. Se escala con el _master id_ del bundle.

Dice:
> The customer encountered a bundle issue with the order. The bundle has been rejected, and an error occurred while attempting to resubmit the order.

> [!Important]
> Cuando hagan referencia a un problema con el _bundle_ están haciendo referencia a algo que pasó con Beluga.
>
> Ver [[OrderlyMeds Glossary]] para ver qué es Beluga.

En este caso, Jaime fue al perfil del CX en Success, copió el "Latest master id" y navegó a la sección "FlaggedRxWrittenBundles". En esa página hizo una búsqueda con el master id copiado y de los resultados elegir el más reciente.

> [!Note]
> Se puede constatar que sea el "latest master id" desde Salesforce yendo al MP más reciente y revisar la columna "Source System ID" en la sección "Clinical Encounters".

Al entrar al detalle del `rx_written_bundles` se ven varias secciones. Para este caso, en la sección "All" había esto:
![[om_9214.01.png]]

Donde, al detallar, se ve que la prescripción no coincide o no incluye la recomendación hecha desde el Med Picker.

## Caso OM-9218 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

> [!Info]
> Esta es la forma de resolver los tickets cuando por alguna razón se necesita reiniciar el proceso para que el CX pueda obtener la prescripción.

> [!Info]
> Este se resuelve accediendo a la consola en prod mediante el Heroku CLI. Hay que correr un comando para crear el nuevo MP del CX.

Al ingresar a la consola de Heroku:
```bash
heroku console -a orderlymeds-production
```

Se pega esta función para luego ejecutar con el correo del CX:
```ruby
def new_member_period(email, checkin_due_date: Date.today + 14.days)
  account = Account.where(email:).sole.salesforce_account
  previous_member_period = account.latest_member_period
  checkin_deadline_date = checkin_due_date + Salesforce::MemberPeriod::CHECKIN_GRACE_PERIOD
  new_lifecycle_stage = (previous_member_period.customer_type == "Employee") ? "NotApplicable" : "Existing"

  Salesforce::MemberPeriod.create!(
    account: previous_member_period.account,
    status: "ReadyForCheckin",
    customer_type: previous_member_period.customer_type,
    customer_lifecycle_stage: new_lifecycle_stage,
    loyalty_points: previous_member_period.loyalty_points,
    checkin_due_date: checkin_due_date,
    checkin_deadline_date: checkin_deadline_date
  )
end
```

Ejemplo de uso:
```ruby
new_member_period("correodelcx")
```

Minutos después aparecerá un nuevo MP en el perfil del CX en Salesforce. Cuando lo verifique respondo al ticket indicando que el "CX is ready for check-in" y se cierra el caso.

## Caso OM-9337 - Stuck in submitted 🟢🟡

Etiquetas: #om_stuck_in_submitted

En este caso vamos al perfil de CX en Ontraport. Cuando es "stuck in submitted" aparece es el enlace a Ontraport en el perfil en Success.

En Ontraport, en la sección "Scripts" se puede ver el script que dice "*Submitted*" en la columna "Outcome".

### Revisar Case Overview

Con el correo nos vamos a la sección "Case Overview" en Success para ver el detalle de este CX.

Aquí nos fijamos y en la pestaña "Care Validate Submissions" saldrá el estado _waiting_for_prescription_.

Lo siguiente será en la consola de Heroku.

### Cambiar el estado del Request en Consola

> [!Warning]
> Los cambios en Ontraport tardan en verse reflejados. La recomendación es dejar un comentario o un recordatorio en alguna parte sobre el ticket y continuar con el resto.
>
> El recordatorio servirá para volver y revisar el estado para ver si se completa como se espera.

Copiamos el ID del request (antes de state que dice _waiting_for_prescription_). En la consola de Heroku:
```ruby
request = CareValidate::Request.find("019eb7a3-8817-79fd-b8d3-c1840b487f8c")
```

Y se cambia su campo `state` a `needs_resubmission`:
```ruby
request.update!(state: "needs_resubmission")
```

> [!Important]
> Esto resulta en un nuevo webhook que creará un nuevo Request.
> Así se ve en el detalle del Care Validate Request:
> ![[om_9337.01.png]]

Después de esto el Care Validate Request cambia visiblemente a "needs resubmission" en Success y hay que ir a Ontraport a hacer el resubmit.

### Resubmit en Ontraport

Pasos desde el script afectado.

1. Cambiar el Outcome a "Active"
2. Vamos al menú "Notes and Tasks" y ubicamos el "Task Manager"
3. Se elige de la lista el indicado (puede haber más de uno) y del menú desplegable "Actions" se escoge _reopen_
	1. Confirma en el modal
4. De nuevo usamos el menú desplegable "Actions" y escoge "Mark complete"
	1. Se abre un nuevo modal
	2. Se espera a que salga el campo "What happened?"
		1. Elige "Approved"
	3. Clica en "Mark Complete & Close"
5. El script cambiará al estado "Active"

### Indicadores de que el resubmit está trabajando ℹ️

En el Case Overview el Request que estaba en _waiting_for_prescription_ se habrá cancelado y habrá uno nuevo.

Otro indicador de que el resubmit funcionó es que el **Outcome** del script cambió a "Pharmacy Selected". También podría estar en "Order At Pharmacy".

## Caso OM-9327 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

Corrí el comando en la consola para crear el nuevo MP. Indiqué en el hilo del Linear y cerré el ticket.

```ruby
def new_member_period(email, checkin_due_date: Date.today + 14.days)
  account = Account.where(email:).sole.salesforce_account
  previous_member_period = account.latest_member_period
  checkin_deadline_date = checkin_due_date + Salesforce::MemberPeriod::CHECKIN_GRACE_PERIOD
  new_lifecycle_stage = (previous_member_period.customer_type == "Employee") ? "NotApplicable" : "Existing"

  Salesforce::MemberPeriod.create!(
    account: previous_member_period.account,
    status: "ReadyForCheckin",
    customer_type: previous_member_period.customer_type,
    customer_lifecycle_stage: new_lifecycle_stage,
    loyalty_points: previous_member_period.loyalty_points,
    checkin_due_date: checkin_due_date,
    checkin_deadline_date: checkin_deadline_date
  )
end

new_member_period("lawrencebmarshall@gmail.com")
```

## Caso OM-9333 - Reset Check-In 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("anita@ctive.net")
```

## Caso OM-9328 - Reset Check-In 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("lenamoyle@gmail.com")
```

## Caso OM-9332 - Reset Check-In 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("ijoguike@gmail.com")
```

## Caso OM-9217 - Reset Check-In 🟡ℹ️

Etiquetas: #om_new_mp #om_checkin_reset

> [!Note]
> Este tiene la particularidad que el MP se quedó en `PharmacyOrderConfirmed`

```ruby
new_member_period("egriffin1789@gmail.com")
```

## Caso OM-9329 - Reset Check-In 🟡ℹ️

Etiquetas: #om_new_mp #om_checkin_reset

> [!Note]
> Este tiene la particularidad que el MP se quedó en `ReadyForProductSelection`

```ruby
new_member_period("glenn.sylvan1@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9330 - Reset Check-In 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("brendaluvschance@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9359 - Reset Check-In 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("marcofalcioni@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9365 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("ibgoogey@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9364 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("simmyunlimited@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9363 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("brittisernia@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9344 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("dcorry73@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9331 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("vanweelden16@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9368 - New MP 🟡

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("melissa.eagar@outlook.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

