# Ciclo 48 OrderlyMeds Support

> [!Info]
> Del Jueves 4 al Miércoles 17 de Junio.

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

## Caso OM-9220 - New MP en Ontraport 🟢

Etiquetas: #om_new_mp #om_new_script #om_ontraport

> [!Important]
> Este tiene dos formas de resolverse. Una es usando el API en Swagger (ver notas en Coshi Notes para acceso a ese servicio). La otra es creando un nuevo Script en Ontraport copiando detalles del Script anterior.

### Swagger

Usa la URL en las notas + las credenciales. Busca el endpoint PUT de "Scripts". Hay dos así que toca mirar el campo "Prescriber".

![[om_9220.01.png]]

Si dice Care Validate, se usa el endpoint correspondiente. La otra opción es Beluga Health.

La petición se hace con el ID del CX en Ontraport y el ID del Script anterior. Está en la URL.

> [!Note]
> Para encontrar el Script previo fijate en el consecutivo de los Scripts.

### Nuevo Script en base al anterior 🔑

Si lo anterior no funciona, se puede crear un Script manualmente usando datos de uno previo.

Pasos a groso modo:

- Copiar el email del CX en el perfil de Ontraport
- Desplaza hacía la sección Scripts y clica en "New Script"
- Clica New Script en el modal
	- Eso nos lleva al form del Script
- Llenar los campos siguientes en base al Script anterior
	- Outcome: Check In
	- Next Consult:
	- Visit Type: weightlossfollowup
	- Current Dosage
	- Prev Length of Vial
	- Prescriber

Con eso en orden toca hacer otras cosas para activar los flujos de cuando se crea un Script en situaciones normales.

#### Verificación con Impersonate

Para confirmar hay que usar el Impersonate del CX. La idea es poder pasar del botón "Check In".

> [!Info]
> Si hay problema, revisa la fecha de Next Consult en el Script.

#### Automations cuando hay errores al probar Impersonate

Prueba primero "Reset Script". Sino funciona. Busca el que es "Create URLs" o algo similar.

En la sección de Automations agrega el que se llama "Reset Script".

xxx

## Caso OM-9337 - Stuck in submitted 🟢

Etiquetas: #om_stuck_in_submitted #om_ontraport

En este caso vamos al perfil de CX en Ontraport. Cuando es "stuck in submitted" aparece es el enlace a Ontraport en el perfil en Success.

En Ontraport, en la sección "Scripts" se puede ver el script que dice "*Submitted*" en la columna "Outcome".

Esto lo solucionamos haciendo dos pasos.

### Paso 1: Revisar Case Overview

Con el correo nos vamos a la sección "Case Overview" en Success para ver el detalle de este CX.

Aquí nos fijamos y en la pestaña "Care Validate Submissions" saldrá el estado _waiting_for_prescription_.

Lo siguiente será en la consola de Heroku.

#### Cambiar el estado del Request en Consola

> [!Warning]
> Los cambios en Ontraport tardan en verse reflejados. La recomendación es dejar un comentario o un recordatorio en alguna parte sobre el ticket y continuar con el resto de casos.
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

### Paso 2: Resubmit en Ontraport

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
5. El script cambiará al estado **"Pharmacy Selected"**

#### Indicadores de que el resubmit está trabajando ℹ️

En el Case Overview el Request que estaba en _waiting_for_prescription_ se habrá cancelado y habrá uno nuevo.

Otro indicador de que el resubmit funcionó es que el **Outcome** del script cambió a "Pharmacy Selected". También podría estar en "Order At Pharmacy".

## Casos de New MP / Check In Reset 🟢

Etiquetas: #om_new_mp #om_checkin_reset

Casos:
- OM-9327
- OM-9333
- OM-9328
- OM-9332
- OM-9330
- OM-9359
- OM-9365
- OM-9364
- OM-9363
- OM-9344
- OM-9331
- OM-9368
- OM-9405
- OM-9403
- OM-9402
- OM-9385

## Caso OM-9217 - Reset Check-In 🟢ℹ️

Etiquetas: #om_new_mp #om_checkin_reset

> [!Note]
> Este tiene la particularidad que el MP se quedó en `PharmacyOrderConfirmed`

```ruby
new_member_period("egriffin1789@gmail.com")
```

## Caso OM-9329 - Reset Check-In - Starter Pack 🟢

Etiquetas: #om_new_mp #om_checkin_reset #om_checkin_starter_pack

> [!Note]
> Este tiene la particularidad que el MP se quedó en `ReadyForProductSelection`

```ruby
new_member_period("glenn.sylvan1@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

> [!Important]
> Aquí había que hacer lo del Starter Pack porque se quedó en el estado mencionado, incluso el nuevo MP

### Reactivando el Starter Pack 🔑

Así:
```ruby
mp = Salesforce::MemberPeriod.find_by(name: "MP-00508397")
mp.update!(customer_lifecycle_stage: "Restarting")

check_in = mp.patient_checkins.last
check_in.update!(is_starter_plan_only: true)
```

Se hace un Impersonate y se revisa que cargue como Starter Pack:
![[starter-pack.png]]

Lo que aparece dentro del recuadro rojo. En la lista de selección debe aparecer alguna opción que sea "starter pack".

## Casos de Oops Error 🟢

Etiquetas: #om_oops_error

Casos resueltos:
- OM-9221
- OM-9225
- OM-9345

### Descripción

Pasa que los CXs intentan acceder al portal y les sale el error:
> Oops! We encountered an error. Our bad.

> [!Important]
> La causa de esto es que el email de la cuenta no está asociado a la cuenta de WorkOS.
>
> Parece pasar más que nada en Salesforce.

En estos casos hay que revisar en consola si la cuenta tiene el campo `workos_user_nk`:
```ruby
account = Account.find_by(email: "")
account.workos_user_nk
```

Si no lo tiene, esto es lo que causa el error en primera medida.

### Actualizando `workos_user_nk`

Hay que ir al perfil en Success, hacer clic en "Impersonate" para llegar a su perfil en WorkOS. En la parte superior se encuentra el ID (debajo del nombre del CX). Es ese el que debemos usar para actualizar este campo.

```ruby
account.update!(workos_user_nk: "")
```

### Actualizar referencia de WorkOS en Salesforce

Finalmente, hay que correr este comando para sincronizar en el lado de Salesforce:
```ruby
Salesforce::CustomerUser.create(
  salesforce_person_account: account.salesforce_account,
  local_account: account
)
```

### Conclusión

Al terminar todo, responder en el hilo con:
```
Issue should be resolved. Please confirm.
```


## Caso OM-9215 - Script error - Beluga Missing Values 🟢ℹ️

Etiquetas: #om_script_error #om_beluga

El caso dice:
> BelugaService: Missing Values

Para llevarlo adelante Pat comentó que:
> Patient confirmed no sensitivity to additives. Resubmitted

Al día siguiente, Yedwy dijo que:
> order at the pharmacy

Y cerró el caso.

## Caso OM-9334 - Check In Error - Heroku Connect 🟢ℹ️

Etiquetas: #om_checkin_error #om_heroku_connect_error

Dice:
> We're having trouble finding your previous medication and are unable to proceed with your order. Please contact support for assistance.

Jaime me explicó que en este caso revisar en Salesforce si el MP tiene "Medication Requests". Si no hay nada, quiere decir que ==hubo un problema en la migración de Ontraport a Salesforce.==

Ver el detalle de cómo resolver en [[Med Requests Import Manual de Ontraport a Salesforce]]

> [!Important]
> Sí se resolvió cómo explicó Jaime. Pasa que el día que me explicó había algún bug y no terminaba bien el proceso. Cuando Fabian probó días después, el modo manual funcionó.

## Casos de Stuck in Submitted 🟢

Casos:
- OM-9389
- OM-9338

Seguí los pasos del caso OM-9337:

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```


## Caso OM-9361 - MP Stuck in ReadyToCreateVisit 🟢

Etiquetas: #om_stuck_in_readytocreatevisit

Hay instrucciones para esto en el Notion no oficial. Son estos tres pasos.

Caso relacionado: [OM-7070](https://linear.app/orderlymeds/issue/OM-7070/rachel-reyes).

### Paso 1: Intenta Crear la Visita

Con esto:
```ruby
mp = Salesforce::MemberPeriod.find_by(omid: "MPOMID")
ce = mp.clinical_encounters.last
BelugaHealth::Scheduler::CreateVisitJob.new.perform(ce.id)
```

El omid se saca en el detalle del MP, pestaña "Details".

Un error que puede dar es este:
```
'BelugaHealth::ApiClient#visit_form_submission': BelugaHealth#visit_form_submission failed with status 400: Patient not eligible for this visitType (BelugaHealth::ApiClient::Error)
```

En ese caso se sigue con el paso 2.

### Paso 2: Escalar a CS

Se manda un mensaje en el hilo del caso a @cs-lead mencionando el error. Ejemplo:
> cx completed and paid for the order, but it is now stuck in "Ready to Create Visit." When trying to create the visit, Beluga returns the error: _"Status 400: Patient not eligible for this visit."_ Please contact the provider to determine why the patient is not eligible for the visit.

Esperar a que ellos solucionen con el proveedor (Beluga en este caso).

> [!Warning]
> Este caso en particular no tuve que llegar al paso 3 porque de CS me dijeron que ya lo atendieron y podía cerrar.

### Paso 3: Intentar crear la visita nuevamente

Vuelve a correr el script:
```ruby
BelugaHealth::Scheduler::CreateVisitJob.new.perform(ce.id)
```

Resultados esperados:
- Se crea la Visita
- El Member Period avanza al siguiente estado
	- Debería ser: `VisitCreated`