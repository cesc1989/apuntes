# Gestión de Pagos y Suscripciones

> [!Important]
> Para todo esto es importante mantener a DevAsPros como SAS activa y con cuenta bancaria. No hacer nada de pagos a nombre propio.
>
> Por temas legales es mejor que todo sea por la SAS.

Apenas empiece fuertemente a abrir enlacito al público voy a optar por dos capaz de oferta:

- Plan Gratuito limitado
	- Acortar hasta 5 enlaces
- Plan Pago ilimitado

Hay dos opciones para esto. Integrar una pasarela de pago (Payment Gateway) o un Merchant of Record. El primero solo se encarga de la parte donde se cobra el dinero dejando los impuestos a cargo del negocio. El segundo se encarga de cobrar y de gestionar los impuestos.

Dado al alcance y esfuerzo limitado que le puedo dedicar al desarrollo tengo que buscar la solución más sencilla y eficiente del momento. Tal como mencionan en Getting Real esa opción puede ser procesar los pagos manualmente o integrar una pasarela de pagos de manera básica.

Esa forma básica es usar Links de Pago. Tales como los ofrece PayPal o Bold.

## Pasarela de Pago (Payment Gateway)

> [!Important]
> Los payment gateways no gestionan impuestos ni retenciones. Queda a cargo del servicio/equipo de desarrollo.

Hay muchas opciones para usar en Colombia. Sin embargo, por la naturaleza de Enlacito, no tiene sentido hacer marketing y SEO solo para Colombia. Debo pensar en global de salida.

Por eso hay que optar por una pasarela que permita recibir pagos internacionales.

Pensaba iniciar con una combinación de pasarela para pagos nacionales y otra para pagos internacionales.

### Pasarelas Nacionales

En primera instancia había pensado en combinar varias pasarelas. Para ello buscaría una que me permitiera aceptar pagos en Colombia y Latam. Podría ser alguna de estas:

- ePayco: https://epayco.com/
- Payu: https://corporate.payu.com/colombia/es/
- Mercado Pago: https://www.mercadopago.com.co/herramientas-para-vender/check-out#benefits-checkout
- Bold: https://bold.co/

Podría elegir una de las anteriores y quedaría cubierto en Colombia. Sin embargo, por lo que explica Freddier, en Colombia (y Latam) hay poca capacidad de pago. Por eso no es conveniente cerrarme solo al mercado nacional.

Enlacito debe arrancar con visión global.

#### ePayco

- En el plan de ePayco + Davivienda: **2.68% + 900 COP x Transacción.**
- En plan otros bancos: **3.29% + 700 COP x Transacción.**

Se agrega un **0.8% para TCs internacionales.**

#### PayU

Para personas o negocios en Colombia: **3.29% + 300 COP x Transacción.**

#### Mercado Pago

Se cobra según cuándo se quiere tener disponible el dinero en la cuenta.

- Al instante: **3.29% + 800 COP + IVA.**
- En 7 días: **2.99% + 800 COP + IVA**
- En 14 días: **2.79% + 800 COP + IVA**

#### Bold 💪🏾

Sitio web: https://bold.co/pagos-en-linea/link-de-pago

Este salió como sugerencia de Kagi Assistant. Sería una forma de empezar con una pasarela nacional que acepta pagos internacionales.

Lo interesante de Bold es que puede cobrarse en USD. Ver: https://ayuda.bold.co/es/preguntas-frecuentes-de-cobro-en-dlares-multimoneda-By6_T2lB1l

##### Tarifas x Desembolso

Por recibir el dinero al día siguiente hábil en cuenta Bold:
- Tarjetas nacionales: **2.99% + 900 COP por transacción.**
- Tarjetas internacionales: **3.99% + 900 por transacción.**
- PSE, Bancolombia y billeteras: **2.89% + 900 por transacción.**

##### Cuenta Digital Bold

En vez de abrir una cuenta bancaria en otro banco, se abre una en Bold por lo cual la plata de los cobros queda en este cuenta. Bold ofrece bolsillos y CDTs para hacer rendir ese dinero.

### Pasarelas Internacionales

Encontré Rebill por recomendación de Kagi Assistant.

#### Rebill

> Collect payments in local currencies across 10+ countries, with deep coverage in Latin America.

Sitio web: https://www.rebill.com/en

Se enfoca más en aceptar pagos en países de Latam pero parece que igual acepta pagos internacionales.

##### Tarifas x Desembolso

Para Colombia. No incluyen IVA ni otras retenciones.

- TC, TD y PSE: **3.29% + 800 COP x Transacción.**
- Tarjetas Internacionales: **4.29% + 800 COP x Transacción.**

## Merchant of Record

Leyendo en Reddit descubrí que existen estos sistemas que son una plataforma encima de pasarelas de pagos. La principal diferencia es que estos se vuelven la cara del negocio en el proceso de pagos. Hacen todo: cobran, tasan impuestos, gestionan devoluciones y disputas, y más.

Son más caras que una pasarela de pagos pero con la ventaja de que se encargan de todo lo fiscal y aceptan pagos internacionales.

Encontré varias:

- Lemon Squeezy: https://www.lemonsqueezy.com/
	- Disponible para Colombia: https://docs.lemonsqueezy.com/help/getting-started/supported-countries#supported-countries-for-merchants-and-affiliates
- Polar: https://polar.sh/
	- Disponible para Colombia: https://polar.sh/docs/merchant-of-record/supported-countries
- Dodo Payments: https://dodopayments.com/
	- Disponible para Colombia???

### Lemon Squeezy 🍋

#### Tarifas x Cobros

Todo sobre las tarifas: https://docs.lemonsqueezy.com/help/getting-started/fees

- Tarifa base: **5% + 0.50 centavos de dólar por transacción.**
	- si el pago es fuera de USA: **+1.5%**
	- si el pago es por PayPal: **+1.5%**
	- si es una subscripción: **+0.5%**

#### Tarifas x Desembolso

Esto es la plata que recibe DevAsPros. En mí caso todo aplica a fuera de USA.

- Mediante Stripe: **1% por cada pago.**
- Mediante PayPal: **3% máximo 30 USD por cada pago.**

### Polar

#### Tarifas x Pagos

- Tarifa base: **4% + 0.40 centavos de dólar por transacción.**
	- si el pago es fuera de USA: **+1.5%**
	- si es una subscripción: **+0.5%**

#### Tarifas x Desembolso

Stripe:
- $2 per month of active payout(s)
- 0.25% + $0.25 per payout

Internacionales:
- 0.25% (EU)
- 1% in other countries

## Decisión Preliminar

### Bold 💪🏾

> [!Warning]
> En este caso tendría que replantear el precio de Enlacito. O lo dejo como está y cubro el IVA o agrego el IVA en el checkout. En este último caso hay que agregar aclaraciones sobre impuestos y tasas.

La implementación sería enviando enlaces de pago la fecha del mes que corresponda. Una vez se complete el pago crear un registro en la tabla `payments` y actualizar la bandera correspondiente en la organización.

## Definitivo al Detalle 🔑

Hoy, creo que arrancar con Bold es una buena opción. Tendría que hacer cosas manuales pero para validar e ir arrancando está bien. No necesito automatizar todo de salida. Si es por los impuestos le pido ayuda a Lariana o busco un contador.

A falta de bocetar y diseño de baja fidelidad, creo que el flujo sería así:

- Cliente se registra con sus datos básicos
- Cuando quiere usar Enlacito más allá de lo base o pagar va a Cuenta -> Facturación
- En esa página mostrar el botón Cambiar a plan Pro
- Se piden más datos para facturar:
	- País, ciudad, dirección, teléfono, correo
	- Lo del país es clave para tener en cuenta lo del IVA
- Se muestra una página:
	- explicando que va a ir al link de pago de Bold o al checkout
	- que el pago tomará algunos minutos en efectuarse
	- si hay demora contactar al correo X mostrando captura de ser posible

En esta parte me toca probar o leer documentación sobre sí se devuelve a la página de origen o si hay webhooks o algo para obtener esa información.

Con el pago confirmado tengo que hacer dos cosas en el sistema:
- Crear un registro para el cliente en la tabla `payments`
	- Esto será una tabla para llevar trazo de los pagos del cliente
	- Así se puede mostrar en una página
	- También para llevar control desde el sistema y no depender de Bold
- Actualizar la bandera correspondiente en el registro de Organization
	- Una bandera de campo tipo entero que indique que es Pro o similar

### Otros Detalles

Cómo usar el ambiente de pruebas de Bold: https://developers.bold.co/pagos-en-linea/boton-de-pagos/ambiente-pruebas