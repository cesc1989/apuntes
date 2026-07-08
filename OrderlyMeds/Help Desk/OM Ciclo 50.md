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

## Caso OM-9804 - MP Stuck on VisitCreated 🟡

> [!Note]
> De este son varios casos similares.

> [!Info]
> Vi que Fabian ubicó el Latest Master ID que es el mismo ID del Clinical Encounter más reciente (cuando el cx está en Salesforce). Con eso fue al canal de Beluga y preguntó sobre el estado de esos master ids.

> [!Info]
> El código de done sale esto está en `Admin::ContactAdapter`. Es la función `latest_master_id`. Ahí se ve que sale de `clinical_encounters` cuando el contacto está en Salesforce.

## Caso OM-9848 - Unable to find matching product 🔵ℹ️

Etiquetas: #om_case_error

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

> [!Info]
> Este es un caso que le tocó a Fabian. Resulta que la medicina elegida fue Wegovy pero esa medicina solo la provee PharmacyHub. Dicha farmacia para estar desactivada en OrderlyMeds.
>
> La solución para ese caso parece será un Cancel & Refund.

## Caso OM-9874 - CC anterior sale como principal el Portal 🟢💳

Etiquetas: #om_remove_credit_card 

El cliente tiene dos tarjetas. Una que ya había cancelado y una nueva. El portal del paciente estaba mostrando la cancelada como por defecto. La solicitud era corregir esto para que la nueva fuera la por defecto.

La solución es ver que `account.salesforce_account.card_payment_methods` devuelve y marcar la errónea como "Inactive". Tal como se hizo en [[OM Ciclo 49#Caso OM-9400 - Remove CC de Salesforce 🟢💳]]