# Dub - Acortador Moderno

> [!Note]
> De salida me llama la atención que la página parece ser de esas de productos de herramientas de desarrollo que son todas pesadas, laggean y parecen estar hechas en la misma plantilla en NextJS o Remix.
> 
> Al menos en Firefox, el registro y la página principal van relaggeados.

> [!Note]
> De hecho se siente así porque está hecha en NextJS. [Ver el repo](https://github.com/dubinc/dub)

## Inicio y Registro

Dub es la versión gringa de Lowerhot. Es un acortador de enlaces potenciado por múltiples características enfocadas en mercadeo.

Muy similar a [[Lowerhot]] en el sentido de que tiene Geo Targeting y soporte para parámetros UTM.

Al registro te pide ingresar un código OTP y luego te lleva por un wizard para completar la configuración de la cuenta.

> [!Important]
> Dub ya tiene configurado el modelo Organization. Al configurar la cuenta se crea una workspace el cual es quien será dueño de los recursos.

### Consideraciones

Se ve que Dub está cargado de características. Por ejemplo:

- Atajos de teclado
- Filtros avanzados
- Analítica avanzada
- Deep linking
- Link cloaking
- Devise targeting
- Webhooks
- Integraciones con otros servicios
- Carpetas
- Link preview customization

## Diseño y Colores

Como mucho otros usa un sidebar en el lado izquierdo. La paleta de colores es bien corta. Se limita a blanco, gris y negro para los botones y opciones de menues.

## Acortando enlaces

![[001.dub.acortando.png]]

Está cargadísimo de salida. Similar a Lowerhot. Esto no es lo que quiero lograr desde el día uno en enlacito. Una gran cantidad de opciones para las personas que están en temas de mercadeo.

![[002.dub.preview.png]]

De salida genera un QR y también tiene personalización de la vista previa cuando se comparte el enlace en redes sociales o aplicaciones de mensajería.

## Filtro de fechas

Este es el más avanzado que he visto por ahora. Creo que es porque debe ser algún componente React gratuito. He visto ya este componente en otras aplicaciones y sitios.

![[003.filters.png]]

# Planes

El plan gratuito tiene un límite bastante modesto.

![[004.free.plan.png]]

Los planes son los más caros que he visto hasta ahora.

![[DevAsPros/Enlacito/descubrimiento/dub/attachments/005.plans.png]]