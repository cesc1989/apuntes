# Configuración de Pagos
En esta sección se pueden configurar varios asuntos pertinentes a pagos de productos.

**Importante**

![](https://paper-attachments.dropboxusercontent.com/s_E790A05EC4FEC0A8BB85872922E00CFA1B85164577CF4B3A04F92E544AEB78E3_1677350452840_imagen.png)

## Proveedores de pagos

Aquí se eligen todos los proveedores para  aceptar métodos de pago como Paypal, Payu, etc.

Estos los configuramos aquí para la tienda pero hay que ir a cada proveedor, registrarnos y obtener los códigos de acceso para poder integrarlos con Shopify.


> Tener en cuenta que ellos cobran sus respectivas comisiones.

En el detalle de cada proveedor disponible y activado se podría activar el modo prueba. También se pueden activar o desactivar franquicias de tarjetas.


> Shopify se encarga de activar los proveedores según la ubicación de la tienda. Esta ubicación es la configurada en “Mercados”

**Parece que solo se puede elegir uno a la vez.**

## Formas de pago adicionales

Otros mecanismos de pago diferentes al proveedor principal. Ejemplo son PayPal o Bitcoin o tarjetas de crédito.

Acá se elige una forma y se conecta mediante un proveedor de pagos.

Ejemplo, en “Proveedores de pagos” elegí Mercado Pago y en “Formas de pago adicionales” puedo elegir Skrill.

Ambas ofrecen lo mismo para el ejemplo pero se puede elegir una combinación de Mercado Pago y Bitcoin.


## Métodos de pago manual

Bastante autodescriptiva

![](https://paper-attachments.dropboxusercontent.com/s_E790A05EC4FEC0A8BB85872922E00CFA1B85164577CF4B3A04F92E544AEB78E3_1677351391666_imagen.png)



# Sobre “Captura de pago”
![](https://paper-attachments.dropboxusercontent.com/s_E790A05EC4FEC0A8BB85872922E00CFA1B85164577CF4B3A04F92E544AEB78E3_1677351450081_imagen.png)


¿Qué significa?

Veamos la [documentación](https://help.shopify.com/es/manual/payments/payment-authorization).

Esta sección nos presenta de opciones de captura: automatica y manual.

**Automática** es como funciona normalmente una plataforma: se verifica la tarjeta y se cobra apenas el proveedor autorice. No hay que hacer más nada por parte de la administración de la tienda.

En cambio **Manual** está pensada para cuando:

- Revisar el análisis para detectar fraudes de un pedido antes de decidir si capturar la transacción o cancelar el pedido. 
- Cumplir con una norma contable que requiere capturar el pago en el momento de la preparación de pedidos. 
- Cumplir con la reglamentación local que prescribe que los comerciantes online ofrezcan al menos una forma de pago que permita a los clientes pagar el pedido en el momento de la entrega. 

Desde mi experiencia como dev no tengo ni idea de cuándo sería manual así que por defecto debería elegir “automática”.

