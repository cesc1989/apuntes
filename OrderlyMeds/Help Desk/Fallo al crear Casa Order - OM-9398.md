# Detalle de todo el flujo del fallo al intentar crear Casa Order

> [!Warning]
> Esto es sobre el problema que hubo con CareValidate y el payload que se enviaba con respecto al Pharmacy y el MedId

> [!Important]
> Este caso ha tenido de todo. Empezó en [[OM Ciclo 49#Caso OM-9398 - Stuck in Active 🟢🔑]] donde Fabian cambió el `strength` en el payload del IncomingWebhook.
> 
> Luego pasó que el `Casa::Order` referenciaba a un `Pharmacy` incorrecto.
> 
> Traté corrigiendo el ID de pharmacy pero eso fue insuficiente porque necesitaba actualizar todo el orden (teniendo en cuenta el `med_id`) para que fuera con los datos necesarios para la Pharmacy.
>
> Sin embargo, Fabian explicó que eso no era correcto. Había que cancelar la orden y hacer resubmit.

Esto es la explicación al detalle de este caso y todo lo que involucra.

## ¿Por qué se reabre?

Comentaron esto:
> I know this has been just resubmitted. However, outcome is prescribed. Bellow is the error

Con esta captura de Ontraport:
![[om_9398_03.png]]

Pregunté a Fabian y me dijo:
> significa que ==esta prescrito pero que falló a la hora de crear la orden==. Es un problema aparte.

## Pistas

Revisando en Case Overview, en la pestaña "Casa Pharmacy Orders" aparece esto:
![[om_9398.04.png]]

Nota el label amarillo que dice "pending_support_review". Esa es la pista que me dio Fabian.

Esto denota un error del sync en `Casa::SubmitCasaOrder`. Snippet:
```ruby
if Casa::ClassifyOrderFailure.permanently_unorderable?(failure_reason)
	casa_order.mark_pending_support_review!
end
```

#### ¿Por qué Casa y no PerfectRx?

Porque cuando se revisa la pestaña "Orders" en Case Overview se ve que está pendiente la de Casa.

![[om_9398.05.png]]

## Revisando Casa Order en la BD

> [!Note]
> El ID de `Casa::Order` está en el registro que aparece en la pestaña "Casa Orders" en Case Overview. Está en fondo gris.

Fabian indica revisar esto en la base de datos usando parte del código de la clase `Casa::SubmitCasaOrder`. Precisamente:
```ruby
casa_order = Casa::Order.find("019ef238-5739-74ce-88df-7ea6526aef27")
failure_reason = Casa::ClassifyOrderFailure.call(casa_order: casa_order, logger: Logger.new($stdout))
```

Eso da una razón del fallo. En este caso fue: `:case_closed`.

> [!Info]
> Si un case está cerrado la API no los acepta y rechaza la petición.

## Pedir reabrir el Case

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

## Comprobar Case está abierto

Con este código se comprueba el estado:
```ruby
casa_order = Casa::Order.find("019ef238-5739-74ce-88df-7ea6526aef27")
failure_reason = Casa::ClassifyOrderFailure.call(casa_order: casa_order, logger: Logger.new($stdout))
```

Debería devolver algún estado que no sea:
```ruby
:no_case
:case_closed
:no_approved_decision
```

Según como indica la clase `Casa::ClassifyOrderFailure`.

## Reenviar el order

Cuando lo anterior esté en buena forma se hace lo siguiente para que se pueda continuar el proceso:
```ruby
casa_order = Casa::Order.find("019ef238-5739-74ce-88df-7ea6526aef27")
casa_order.reset!
Casa::SubmitCasaOrder.call(casa_order: casa_order, logger: Rails.logger)
```

# Escalar a Care Validate

Desde el equipo de `#om-carevalidate-patient-exp` dijeron que el caso estaba abierto.

Así que escalamos a `#ext-carevalidate`. Pregunté sobre cómo se sabe si está abierto y si era posible limpiar el campo `closedAt`.

> [!Important]
> Desde CV dicen que el caso se sabe si está abierto por el campo `isArchived`:
>
> > The case is in the open state. Can be verified using `case.isArchived` value in the payload for `/api/v1/customer-case-detail`.

## Sistema usa `closedAt` para indicar Care Validate `case_closed`

Para comprobar el estado del Case se una petición al API de Care Validate:
```ruby
CareValidate::Api::Case.fetch_by_case_id(case_id: casa_order.case_nk)
```

Lo que da una respuesta que incluye estos valores:
```ruby
"status" => "APPROVED",

"createdAt" => "2025-11-23T23:17:42.590Z",
"updatedAt" => "2026-06-23T15:33:30.873Z",

"inProgressAt" => "2025-11-23T23:17:43.546Z",
"closedAt" => "2026-06-23T02:03:48.967Z",
"isArchived" => false,

"closedBy" => "lfnptZe8BVZRMiKm2r5dv1pu7fN2",
"inProgressBy" => "62bc7b9c-cf85-47dd-947f-464559eb7a7a",
```

Si bien el campo `status` está en "APPROVED" la función de `Casa::SubmitCasaOrder` sigue devolviendo `:case_closed`.

Esto es porque en el llamado a `Casa::ClassifyOrderFailure.call(casa_order: casa_order, logger: logger)` la función evalúa si hay algún valor en `closedAt`:
```ruby
module Casa
  class ClassifyOrderFailure
	  def call
		  detail = CareValidate::Api::Case.fetch_by_case_id(case_id: casa_order.case_nk)
      return :no_case if detail.blank?
      return :case_closed if detail["closedAt"].present?
	  end
  end
end
```

# Cancelar Order y Hacer Resubmit

El fix de esto no es actualizar el pharmacy_id correcto. Lo que toca hacer es cancelar la orden con problema y hacer un resubmit en Ontraport. Para ello hay que buscar el último webhook desde Ontraport y ejecutarlo de nuevo.

Hay una clase que se llama `CareValidate::FixResubmitOntraportWebhook` que hace todo esto siempre y cuando el último `IncomingWebhook` sea `source_type: "ontraport"`. Si no lo es, toca ejecutar el código de manera manual.

Así sería la cosa cuando el webhook más reciente no es de Ontraport:
```ruby
request = CareValidate::Request.find("019e9877-d0fd-7660-ad53-2793fd8301a3")
icwhs = IncomingWebhook.where(id: request.incoming_webhook_ids)

result = CareValidate::FixResubmitOntraportWebhook.call(
  request_id: "019e9877-d0fd-7660-ad53-2793fd8301a3"
)
puts result
# {error: "Latest incoming webhook must be from Ontraport"}
```

Entonces toca hacer la ejecución manual.

## Correr FixResubmitOntraportWebhook manualmente

Paso a paso.

```ruby
request = CareValidate::Request.find("019e9877-d0fd-7660-ad53-2793fd8301a3")

ontraport_webhook = IncomingWebhook
  .where(id: request.incoming_webhook_ids)
  .where(source_type: "ontraport")
  .order(created_at: :desc)
  .first

puts "Webhook ID: #{ontraport_webhook.id}"
puts "Source event: #{ontraport_webhook.source_event}"
puts "Source type: #{ontraport_webhook.source_type}"
puts "Created at: #{ontraport_webhook.created_at}"

# Confirma que es el correcto y luego replica lo que hace FixResubmitOntraportWebhook con ese webhook:

# Reset del request
request.update!(
  decision: nil,
  state: "needs_requested_medpicker_data",
  requested_medpicker_data: nil,
  prescribed_medpicker_data: nil
)

# Re-fetch MedPicker
medpicker_api_payload = MedPicker::Ontraport::CreateApiData.call(
  payload: ontraport_webhook.data,
  source_event: ontraport_webhook.source_event
)
requested_medpicker_data =
  Omfs::MedPickerApi
    .new
    .get_med_picker_recommendation_patient_preferences_cheapest_care_validate(
      med_picker_patient_preferences_request_command: Omfs::MedPickerPatientPreferencesRequestCommand.new(medpicker_api_payload)
    )

puts "MedPicker pharmacy: #{requested_medpicker_data&.remote_pharmacy_name}"
puts "MedPicker remoteProductIdentifierNk: #{requested_medpicker_data&.remote_product_identifier_nk}"

# Verifica que remote_pharmacy_name sea EVO antes de continuar. Si es correcto:

request.update!(requested_medpicker_data:)
request.ready_for_care_validate_submission!

CareValidate::SendCheckinJob.new.perform(ontraport_webhook.id, request.id, request.script_nk)
```


### Verification después de resubmit

Tanto en Requested y Prescribed debe dar el mismo nombre.

```ruby
request.reload
puts "State: #{request.state}"
puts "Requested pharmacy: #{request.requested_medpicker_data&.dig("remotePharmacyName")}"
puts "Prescribed pharmacy: #{request.prescribed_medpicker_data&.dig("remotePharmacyName")}"
puts "Decision: #{request.decision}"
```