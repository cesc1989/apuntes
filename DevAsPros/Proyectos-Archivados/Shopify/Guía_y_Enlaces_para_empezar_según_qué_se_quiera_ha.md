# Guía y Enlaces para empezar según qué se quiera hacer
Como ya no están los cursos de App Development ni Theme Development en la Academia, entonces me toca valerme de la documentación de desarrolladores.

En este documento pongo datos y enlaces para saber por donde empezar en el momento que quiera estudiar alguna de las formas de hacer negocios con Shopify.


# Theme Development

Enlaces:

- Página de Themes → https://shopify.dev/docs/themes
- Resumen general → https://shopify.dev/docs/themes/getting-started

Para tener en cuenta:

- Liquid → https://shopify.dev/docs/api/liquid
- Dawn → https://shopify.dev/docs/themes/tools/dawn
- CLI for Themes → https://shopify.dev/docs/themes/tools/cli
- Arquitectura de una plantilla → https://shopify.dev/docs/themes/architecture

¿Qué son los temas en Shopify?

- Los temas determinan la forma en que una tienda en Shopify luce, se siente y funciona para los vendedores y compradores.
- Se construyen usando Liquid, además de HTML, CSS y JavaScript.

¿Qué salida tiene como desarrollador?

- Se puede crear temas personalizados para clientes particulares
- Personalizar temas para necesidades puntuales
- Crear temas y venderlos en la tienda de Shopify
- También se pueden crear apps para extender la funcionalidad de los temas


# App Development

Enlaces:

- Página docs de Apps → https://shopify.dev/docs/apps
- Resumen general → https://shopify.dev/docs/apps/getting-started

Para revisar:

- Shopify CLI for Apps → https://shopify.dev/docs/apps/tools/cli

¿Para qué una app en Shopify?

- Conectar con Shopify para leer o guardar información cuando un usuario ingrese un formulario o en respuesta a un webhook
- Extender las funcionalidades del Admin de Shopify o Shopify Point of Sale (POS)
- Mejorar la forma en que las tiendas muestran la información a sus clientes

¿Cómo empezar?

- Tuto para crear esqueleto app → https://shopify.dev/docs/apps/getting-started/create#step-1-create-a-new-app
- Tuto para hacer la app con Remix → https://shopify.dev/docs/apps/getting-started/build-qr-code-app?framework=remix

Avanzado

- Autenticación y Autorización → https://shopify.dev/docs/apps/auth
- Despliegue de apps → https://shopify.dev/docs/apps/deployment


# Custom Storefront

Enlaces:

- Página principal → https://shopify.dev/docs/custom-storefronts
- Resumen general → https://shopify.dev/docs/custom-storefronts/getting-started

Para tener en cuenta:

- Hydrogen demo store → https://github.com/Shopify/hydrogen/tree/main/templates/demo-store


## ¿Qué son las Custom Storefronts?

Son tiendas cuya experiencia visual se crea con herramientas diferentes a las de Shopify pero el backend es el de Shopify.

Es usar Shopify de manera headless.

¿cuándo crear un *custom storefront?*

- Estás creando una experiencia única que no se puede lograr con las herramientas de Shopify
- Tienes un stack frontend que no soporte Liquid
- Quieres integrar el backend de Shopify en una infraestructura existente

Ejemplos:

- Vender productos en aplicaciones móviles nativas
- Vender productos en experiencias de Realidad Aumentada o Realidad Virtual
- Vender productos en transmisiones de vídeo
- Vender productos mediante un botón de compra en un sitio web existente

