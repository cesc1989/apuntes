# Gestión de Pagos y Suscripciones

> [!Importante]
> Para todo esto es importante mantener a DevAsPros como SAS activa y con cuenta bancaria. No hacer nada de pagos a nombre propio.
>
> Por temas legales es mejor que todo sea por la SAS.

Apenas empiece fuertemente a abrir enlacito al público voy a optar por dos capaz de oferta:

- Plan Gratuito limitado
	- Acortar hasta 5 enlaces
- Plan Pago ilimitado

Dado al alcance y esfuerzo limitado que le puedo dedicar al desarrollo tengo que buscar la solución más sencilla y eficiente del momento. Tal como mencionan en Getting Real esa opción puede ser procesar los pagos manualmente o integrar una pasarela de pagos de manera básica.

Esa forma básica es usar Links de Pago. Tales como los ofrece PayPal o Bold.

## Pasarela de Pago (Payment Gateway)

Hay muchas opciones para usar en Colombia. Sin embargo, por la naturaleza de Enlacito, no tiene sentido hacer marketing y SEO solo para Colombia. Debo pensar en global de salida.

Por eso hay que optar por una pasarela que permita recibir pagos internacionales.

Pensaba iniciar con una combinación de pasarela para pagos nacionales y otra para pagos internacionales. El aspecto nacional queda cubierto en la siguiente sección. Para lo internacional debo optar por una opción diferente o un Merchant of Record.

Trataré de explicar todas las opciones.

### Pasarelas Nacionales

En primera instancia había pensado en combinar varias pasarelas. Para ello buscaría una que me permitiera aceptar pagos en Colombia y Latam. Podría ser alguna de estas:

- ePayco: https://epayco.com/
- Payu: https://corporate.payu.com/colombia/es/
- Mercado Pago: https://www.mercadopago.com.co/herramientas-para-vender/check-out#benefits-checkout

Podría elegir una de las anteriores y quedaría cubierto en Colombia. Sin embargo, por lo que explica Freddier, en Colombia hay poca capacidad de pago. Por eso no es conveniente cerrarme solo al mercado nacional.

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

### Pasarelas Internacionales

Encontré Rebill por recomendación de Kagi Assistant.

#### Rebill

> Collect payments in local currencies across 10+ countries, with deep coverage in Latin America.

Sitio web: https://www.rebill.com/en

Se enfoca más en aceptar pagos en países de Latam pero parece que igual acepta pagos internacionales.

##### Tarifas x Cobros

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

### Lemon Squeezy

#### Tarifas x Cobros

Todo sobre las tarifas: https://docs.lemonsqueezy.com/help/getting-started/fees

- Tarifa base: **5% + 0.50 centavos de dólar por transacción.**
	- si el pago es fuera de USA: **+1.5%**
	- si el pago es por PayPal: **+1.5%**
	- si es una subscripción: **+0.5%**

#### Tarifas x Pagos

Esto es la plata que recibe DevAsPros. En mí caso todo aplica a fuera de USA.

- Mediante Stripe: **1% por cada pago.**
- Mediante PayPal: **3% máximo 30 USD por cada pago.**

## Decisión Preliminar

### Bold

### Lemon Squeezy

## Definitivo al Detalle