# Apuntes del Onboarding en OrderlyMeds

Apuntes de la reunión con Jaime y Fabian.

> [!Important]
> Todos los CX existen en Ontraport. Hay una migración activa de Ontraport a Salesforce. Esto significa que no todos los cx existen en Salesforce.

> [!Info]
> Muchos casos se resuelven por consola de Heroku. O sea, modificando datos o disparando jobs.
>
> Desde el perfil del Customer en Salesforce también se pueden dispara tareas de corrección.

## Sobre Member Period

- El equipo de Customer Support lo llama MP.
- Solo existen en Salesforce.


# Sobre Script

- Solo existe en Ontraport.

## Casos de Stuck In / Script error

Reportes de algo "stuck in" o "script" son de problemas en Ontraport.

### Para los casos de Stuck In

Copiar el correo y buscarlo en la sección "Case Overview". En la pestaña "Care Validate Submissions" se pueden ver los estados del flujo.

#### Si el estado es `waiting_for_prescription`

> [!warning]
> Jaime hizo una grabación de este paso. Ver y tomar apuntes de eso. Onboarding 1. Primera grabación.

- haz clic en el ID linkeado a la vista "Care Validate Request"
	- Ahí se puede ver todo el flujo.
	- Si no hay nada extraño, indicar al Rep que debe contactar con Care Validate.


#### Si el estado es `routed_to_beluga`

Es otro caso. Hay que revisar el flujo del state machine y los datos en la base de datos.

Etiquetas: #om_beluga #om_stuck_in_submitted 

En Ontraport no suele haber muchas pistas. En este caso la pista es que el campo "Prescriber" (perfil del paciente) dice "Beluga Health".

Revisamos en Care Validate Request y vemos el Submission que dice "routed_to_beluga". Si vemos esto tenemos que entender una pieza del código fuente.

El modelo `CareValidate::Request` tiene un state por donde empezamos a revisar. Es `send_to_beluga`. Buscamos dónde se usa ese cambio de estado y encontramos el job `CareValidate::FindOrCreateCaseJob` donde estas tres líneas del rescue llaman la atención:
```ruby
request.send_to_beluga!

::CareValidate.retry_via_beluga(contact: ::Ontraport::Meta::Contact.get_by_id(script.contact))

ProcessIncomingWebhookJob.perform_async(incoming_webhook_id)
```

La primera cambia el estado del request. La segunda cambia el Prescriber en Ontraport. Y la última es la ejecución que nos interesa. Sin embargo, esa ejecución no se da.

Hay que revisar el Request y el IncomingWebhook.

##### Revisando IncomingWebhook para caso de Beluga

Al ver el webhook al detalle podemos ver que los campos `source_event` y `source_type` nos dan una pista:
```ruby
state: "delivered"
source_type: "ontraport"
source_event: "initial_signup"
```

Ya que el campo `state` está en `delivered` el webhook no volverá a hacer su parte. ¿Por qué? Porque en la clase (la de la tercera línea) `ProcessIncomingWebhookJob` tiene un guard que termina el proceso si el estado del webhook es `delivered`:
```ruby
# app/sidekiq/process_incoming_webhook_job.rb

if incoming_webhook.delivered?
	logger.info at: "skipped", reason: "already_delivered"
	return
end
```

La gracia aquí entonces es cambiar el `state` del incoming webhook a un valor que no sea `delivered` para que el job lo pueda procesar.

```ruby
incoming_webhook.update!(state: "processing")
```

Y luego se vuelve a correr el job:
```ruby
ProcessIncomingWebhookJob.new.perform(incoming_webhook.id)
```

##### Verificación de éxito

El Outcome del Script cambia a "Pharmacy Selected". Después de esto ya se puede notificar que el proceso está listo para continuar.