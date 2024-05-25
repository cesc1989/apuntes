# Configuración de Datos Personalizados - metafields
Sobre los metacampos la [documentación](https://help.shopify.com/es/manual/custom-data/metafields/) nos dice que:

> Los metacampos te ayudan a personalizar la funcionalidad y el aspecto de la tienda Shopify, ya que te permiten guardar información especializada que, normalmente, no se captura en el panel de control de Shopify.
> 
> Puedes usar metacampos para hacer seguimiento de manera interna o mostrar información especializada en la tienda online de varias maneras.

Estos sirven para crear definiciones adicionales de cada producto.

Por ejemplo, normalmente en Shopify hay nombre, descripción, precio, código de barras, SKU.

Si yo tengo una tienda de computadores, tendría que decir la cantidad de RAM en la descripción y subrayarla o ponerle negrita. Con los metacampos puedo crear una propiedad que sea “RAM” y en la página de producto darle un valor y este atributo se puede usar para diseñar la página de producto.

Ejemplo en la tienda de prueba: “cantidad de hojas”

![](https://paper-attachments.dropboxusercontent.com/s_2231C4E07941BE5FD4F09422EE1B1BAF8AF7C75B172E1E25B519E5C78B91D45D_1677342026831_imagen.png)


Y luego en el producto puedo dar un valor:

![](https://paper-attachments.dropboxusercontent.com/s_2231C4E07941BE5FD4F09422EE1B1BAF8AF7C75B172E1E25B519E5C78B91D45D_1677342045441_imagen.png)


Pero esto solo es para crearlos y definirlos, aún no se muestran en el tema.

## Mostrando metacampos en la tienda

[Documentación](https://help.shopify.com/es/manual/custom-data/metafields/displaying-metafields-on-your-online-store).

Primero hay que [activarlos](https://help.shopify.com/es/manual/custom-data/access-options) en la sección “Opciones de acceso” cuando se está editando un metacampo.

![](https://paper-attachments.dropboxusercontent.com/s_2231C4E07941BE5FD4F09422EE1B1BAF8AF7C75B172E1E25B519E5C78B91D45D_1677342394748_imagen.png)


Para mostrarlo [editar el tema](https://help.shopify.com/es/manual/custom-data/metafields/displaying-metafields-on-your-online-store#part-d959eaa920d7478b) es una opción.

Para editar el tema ([docs](https://help.shopify.com/es/manual/online-store/themes/theme-structure/extend/edit-theme-code)) vamos al menú “Tienda online” y buscamos la opción “acciones” o el menú de los tres puntos. Ahí elegimos “Editar código”

![](https://paper-attachments.dropboxusercontent.com/s_2231C4E07941BE5FD4F09422EE1B1BAF8AF7C75B172E1E25B519E5C78B91D45D_1677342816044_Screenshot+2023-02-25+at+11.32.43+AM.png)


En la tienda de prueba, la plantilla de producto se llama `product-template.liquid`.

Y se accede al metacampo usando Liquid:

    <p>Cantidad de hojas {{ product.metafields.custom.cantidad_de_hojas }}</p>

Y así se logra mostrar en la plantilla.

Enlaces:

- Documentación de [Metafield en Liquid](https://shopify.dev/docs/api/liquid/objects/metafield)
- [Truthy and falsy](https://shopify.dev/docs/api/liquid/basics#truthy-and-falsy) en Liduiq

