# Detalle de todo el flujo del fallo al intentar crear Casa Order

Esto es la explicación al detalle de este caso y todo lo que involucra.

## ¿Por qué se reabre?

Comentaron esto:
> I know this has been just resubmitted. However, outcome is prescribed. Bellow is the error

Con esta captura de Ontraport:
![[om_9398_03.png]]

Pregunté a Fabian y me dijo:
> significa que ==esta prescrito pero que falló a la hora de crear la orden==. Es un problema aparte.

## Pistas

Revisando en Care Overview, en la pestaña "Casa Pharmacy Orders" aparece esto:
![[om_9398.04.png]]

Nota el label amarillo que dice "pending_support_review". Esa es la pista que me dio Fabian.

Esto denota un error del sync en `Casa::SubmitCasaOrder`. Snippet:
```ruby
if Casa::ClassifyOrderFailure.permanently_unorderable?(failure_reason)
	casa_order.mark_pending_support_review!
end
```

#### ¿Por qué Casa y no PerfectRx?

Porque cuando se revisa la pestaña "Orders" en Care Overview, para este caso, se ve que está pendiente la de Casa.

![[om_9398.05.png]]

## Revisando la Orden en la BD

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

## Pedir reabrir el case

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

## Comprobar case está abierto

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

## Reenviar el order

Para que se pueda continuar el proceso:
```ruby
casa_order = Casa::Order.find "019ef238-5739-74ce-88df-7ea6526aef27"
casa_order.reset!
Casa::SubmitCasaOrder.call(casa_order: casa_order, logger: Rails.logger)
```