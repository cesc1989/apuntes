# OM Ciclo 50

> [!Info]
> Del Jueves 2 de Julio al Miércoles 15 de Julio.

## Caso OM-9839 - New MP diferente dosis 🟢ℹ️

Etiquetas: #om_new_mp_different_dosage

Dice:
> This patient mistakenly filled out the current dosage (first check in) and ended up being prescribed with 2.5mg when been on 15mg.
>
> reset link so can check in asap for 15mg.

Había creado un nuevo MP pero, dicen, quedó con la misma dosis de 2.5mg.

> [!Note]
> Hay instrucciones en el Notion sobre algo relacionado con el MedPicker Recommendations asociadas en Ontraport pero no me queda claro. Pregunté a Wilkes.
>
> Me dijo que:
> > You don't have to do that. You should reset the checkin so that the customer reorder through the normal checkin process.

### Nuevo MP y que el CX elija la dosis en el Check In

Es como explica Wilkes. En el Check In hay una pregunta sobre medicina y dosis. Ahí se puede elegir la de 15mg.

![[om_9839.01.png]]

Así que hice un nuevo MP, revisé con la suplantación y luego informé en el hilo.

## Caso OM-9790 - Script error con `needs_requested_medpicker_data` 🟢ℹ️

Etiquetas: #om_needs_requested_medpicker_data

> [!Important]
> El problema fue porque el request más reciente no tenía el MedID esperado.
>
> Jaime lo corrigió asignando el ID del request cancelado que ya tenía el MedID prescrito.
>
> El MedID es el campo `remoteProductIdenfitifierNK` que está en el hash que devuelve `request.requested_medpicker_data`. Ese valor lo ingresó en el campo que tiene el botón "Fix Medpicker Selection"

Caso de Stuck in Submitted que al hacer el resubmit el Script queda en Outcome "Script Error" y el CareValidate::Request en estado `needs_requested_medpicker_data`.

En Ontraport, en la sección "OMFS Data" del Script dice:
> No matching recommendations for these patient preferences.

La solución fue copiar el MedId y correr el "Fix Medpicker Selection" del Request.

![[om_9790.01.png]]

### Nada de esto sirvió

Intenté correr:
```ruby
result = CareValidate::FixResubmitOntraportWebhook.call(
  request_id: "019f23ba-6d29-7dac-818a-af2c6b672855"
)
```

pero devolvió el error que aparece en Success:
```ruby
{error:
  "No MedPicker recommendation found for CareValidate for patientPreference: [{\"name\" => \"OM Ultra: Tirzepatide - $399/month - 4 Injections\", \"numberMonths\" => \"1\", \"strength\" => \"None\", \"change\" => \"Weight Loss is Going Great\", \"ingredients\" => \"B3,B5\", \"titrateup\" => \"I want to increase dosage every month\", \"medIdOverride\" => \"\", \"refills\" => \"0\"}]"}
```

Luego intenté correr:
```ruby
CareValidate::GetMedpickerDataJob.new.perform(
  webhook.id,
  request.id,
  request.script_nk
)
```

Pero también dio error:
```
No matching recommendations for these patient preferences.
```

> [!Note]
> Para poder correr `CareValidate::GetMedpickerDataJob` el campo `source_event` del webhook debe ser `care_validate_checkin` para que al llamar a `CareValidate::ProcessRequestJob` se ejecute la parte en que se encola a `CareValidate::SendCheckinJob`. Job que también es llamado en el proceso `FixResubmitOntraportWebhook`.

## Caso OM-9804 - Auto MP Stuck on VisitCreated 🔵

> [!Important]
> Ahora estos casos serán asignados automáticamente a Briggs porque se pueden solucionar por Salesforce.

> [!Note]
> De este son varios casos similares porque son auto generados por el sistema.

> [!Info]
> Vi que Fabian ubicó el Latest Master ID que es el mismo ID del Clinical Encounter más reciente (cuando el cx está en Salesforce). Con eso fue al canal de Beluga y preguntó sobre el estado de esos master ids.

> [!Info]
> El código de done sale esto está en `Admin::ContactAdapter`. Es la función `latest_master_id`. Ahí se ve que sale de `clinical_encounters` cuando el contacto está en Salesforce.

## Caso OM-9848 - Unable to find matching product 🔵ℹ️

Etiquetas: #om_case_error #om_unable_to_find_matching_product

> [!Info]
> Este es una continuación de los casos donde tocaba hacer cancel & refund porque estaban con el plan de 3 meses. Hicieron el respectivo C&R pero hicieron resubmit del mismo MP.

Este es el error:
![[om_9848.png]]

Fabian dijo que tenían que tenían que cancelar y devolver o dar créditos para luego crear un nuevo Member Period.

## Caso OM-9722 - Botón Continue bloqueado en Check In 🟢ℹ️

Etiquetas: #om_checkin_blocked

Caso en el que un CX iba a hacer el check in y respondía todo pero el botón de continuar quedaba bloqueado:
![[om_9722.checkin.blocked.png]]

Hice la suplantación y pasaba igual. Pregunté a Fabian y me dijo que, en Ontraport, en la lista de sensitivities, marcara que "No". Eso desbloqueaba el botón.

Eso hice y se desbloqueó al probar la suplantación de nuevo.

## Caso OM-9875 - Not at the Pharmacy - Wegovy 🔵ℹ️

Etiquetas: #om_not_at_pharmacy 

> [!Info]
> Este es un caso que le tocó a Fabian. Resulta que la medicina elegida fue Wegovy pero esa medicina solo la provee PharmacyHub. Dicha farmacia para estar desactivada en OrderlyMeds.
>
> La solución para ese caso parece será un Cancel & Refund.

## Caso OM-9874 - CC anterior sale como principal el Portal 🟢💳

Etiquetas: #om_remove_credit_card 

El cliente tiene dos tarjetas. Una que ya había cancelado y una nueva. El portal del paciente estaba mostrando la cancelada como por defecto. La solicitud era corregir esto para que la nueva fuera la por defecto.

La solución es ver que `account.salesforce_account.card_payment_methods` devuelve y marcar la errónea como "Inactive". Tal como se hizo en [[OM Ciclo 49#Caso OM-9400 - Remove CC de Salesforce 🟢💳]]

## Caso OM-9949 - Beluga Missing Values 🟡

Etiquetas: #om_beluga 

Vi lo mismo que en el caso [[OM Ciclo 49#Caso OM-9483 - Beluga Missing Values 🔵]] (sin sensitivities) así que hice la misma pregunta.

### Comentarios de Fabian 🗣️

Me dijo que en estos casos hay algunas opciones para solucionar:

> 1. Cuando hayan las reuniones con Briggs debe preguntar si pueden confirmar eso sensitivities y una vez confirmado darle resubmit.
> 2. Si por alguna razón le dicen que ellos no deben modificar esos campos desde OP, hablar con Jay (o inclusive la misma reunion con Briggs) y pedirles si pueden darle un refund or credits y que te envíen nuevamente el link del HS para que el cliente lo llene de nuevo.
> 3. Hablar con Wilkes y preguntarle si es posible migrar ese cliente a Salesforce y desde allí hacer el checkin, también requiere refund o credits en salesforce después de migrarse.



## Casos de error de CV enviando el Med a una Farmacia incorrecta 🟢

Etiquetas: #om_not_at_pharmacy

> [!Info]
> Estos son lo mismo que pasó en el caso descrito en [[OM Ciclo 49#Caso OM-9398 - Fallo al crear Casa Order 🟢🔑]]

> [!Important]
> Cuando el Script pasa a "Pharmacy Selected" se puede dar por corregido.
> Sin embargo, estos son solucionados definitivamente es que el Script pasa a "Order at Pharmacy".

Casos:
- **OM-9966** 🟢
	- Resultado final Script en "Order at Pharmacy"
	- Para este solo hice el resubmit normal.
	- Tanto _Requested Medpicker Data_ como _Prescribed Medpicker Data_ están en CasaPharmaRx.
	- El `CareValidate::Request` está en estado `written` con `decision` en "approved".
	- La orden en Casa está con los estados Written, Sent y Received están chuleados en verde.
- **OM-9893** 🟢
	- Hice resubmit pero se quedó en "waiting for prescription"
	- Según Claudio, no hay un tiempo claro sobre cuándo esto debe pasar
		- Solo hay controles por si se pasa de las 48 horas en este estado
	- A este fue al que le hice "Resubmit Latest Ontraport Webhook" y también "Fix Medpicker Selection" pero sigue demorado en el estado Pharmacy Selected
	- ==Volví a hacer el resubmit. Esta vez cuando quede en Pharmacy Selected voy a revisar el estado del CX y el estado de la farmacia del _Requested Medpicker Data_==
	- De nuevo en Pharmacy Selected. CX en OH. MedId de Evoluciona en OH.
	- Le hice "Resubmit Latest Ontraport Webhook" directo desde código.
	- Luego de la demora el mensaje de alerta dice:
		- _CareValidate request awaiting prescription response. Request for script 794134 has been waiting for prescription for 2.9 days. CareValidate has not sent an ADD_CASE_DECISION webhook. Contact CareValidate to check case [ID]. The prescriber may not have reviewed the case, or there was a technical issue on their end._
	- Parece que "Pharmacy Selected" también es un buen estado. Briggs cerró el caso.
- **OM-9950** 🟢
	- Resubmitted
	- Ahora está en Pharmacy Selected
	- Volví a hacer el resubmit.
	- De nuevo en Pharmacy Selected. CX en OH. MedId de Evoluciona en OH.
	- Lo mismo que 9893.
	- Parece que "Pharmacy Selected" también es un buen estado. Briggs cerró el caso.
- **OM-9958** 🟢
	- Resubmitted
	- Ya está en Order At Pharmacy
- **OM-9980** 🟢
	- Resubmitted
	- Ahora está en Pharmacy Selected
	- Volví a hacer el resubmit.
	- De nuevo en Pharmacy Selected. CX en ND. MedId de Evoluciona en ND.
	- Ya aparece como "Order at Pharmacy" y la orden está pendiente de Shipping.
- **OM10001** 🟢
	- Resubmitted
	- Ahora está en Pharmacy Selected
	- Volví a hacer el resubmit.
	- De nuevo en Pharmacy Selected. CX en OH. MedId de Evoluciona en OH.
	- Ya aparece como "Order at Pharmacy" y la orden está pendiente de Shipping.



## Caso OM-9998 - Prescrito en Beluga pero nada en Pharmacy 🟢ℹ️

Etiquetas: #om_beluga_not_at_pharmacy

Dice el caso:
> Prescribed by Beluga on 7/4. All bundles were approved. Still, nothing at the pharmacy.

Tengo que hacer Resubmit to MSO. Cuando voy a hacerlo veo que cambia el medicamento:
![[om_9998.01.png]]

Jaime me dice que en el campo override el formulario ponga el mismo MedID que ya está prescrito.

Cuando lo hago así no cambia el medicamento:
![[om_9998.02.png]]

Y así se indica al completar:
![[om_9998.03.png]]

### Cambios de Estado del Member Period

Después del resubmit pude ver los siguientes cambios de estado del Member Period.

- Pasó a `ReadyToCreateVisit`
- Al minuto pasó a `Visit Created`
- A los tres minutos pasó a `VisitCompleted`
- A los 30 minutos pasó `PrescriptionWritten`
- Para confirmar que quedó bien en algún momento del día debe pasar a `WaitingOnPharmacyConfirmation` o `PharmacyOrderConfirmed` y los siguientes.

Además, el primer Clinical Encounter fue cancelado, se creó uno nuevo que pasó a estar en *Finished*. Lo mismo para Med Picker Recommendations. Se canceló el primero, se creó uno nuevo que quedó en estado *Recommendation Made*.

Actualizaciones:
- 13 de Julio: seguía en `PrescriptionWritten`
- 14 de Julio: pasó a `PharmacyOrderConfirmed`
	- Ya pasó a `PharmacyOrderShipped`
	- Se completó.


## Caso OM-10040 - Success y GHL unlinked 🟢

Etiquetas: #om_ghl_not_linked

La cuenta de GHL no estaba enlazada al Account del CX. El enlace a este servicio se da con los campos:
```ruby
ghl_contact_nk: "",
ghl_contact_url: "",
```

Para enlazarlos se corre el job que ya existe pasando el ID de la cuenta:
```ruby
Salesforce::BackfillGhlContactInfo.new.perform(account.id)
```

### Verificación

Con esto verifico que existan ambas cuentas y que el correo coincida:
```ruby
account = Account.find("019f4865-3387-7121-9ba8-6c18d0f89c48")
puts "Success email: #{account.email.inspect}"
puts "Success ghl_contact_nk: #{account.ghl_contact_nk.inspect}"
puts "Success ghl_contact_url: #{account.ghl_contact_url.inspect}"

ghl_contact = Ghl.client.get_contact(id: "UVLGB6jqO280phI97Gul")
puts "GHL contact email: #{ghl_contact.dig("contact", "email").inspect}"
puts "GHL contact locationId: #{ghl_contact.dig("contact", "locationId").inspect}"
```

- Si `account.email` y `ghl_contact.dig("contact","email")` no coinciden (mayúsculas, typo, email distinto), esa es la causa raíz — el job automático `Salesforce::BackfillGhlContactInfo` busca por email exacto (`email.downcase`, ver `lib/ghl/api_client.rb`), así que un mismatch hace que nunca los enlace.
