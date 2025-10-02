# 001 - Mejorar Carga de Fotos de Platos

Las fotos de los platos de cada menú están cargadas en un bucket en S3. Ahora mismo no hay límite de tamaño ni formato. Si se carga una imagen de 10mb, al cargar el menú toca descargar esos 10mbs.

Estamos en etapa temprana pero una de las cosas que queremos con este proyecto es que el menú sea de verdad bueno. Una de las formas de hacerlo muy bueno es que cargue rápido.

Para hacerlo cargar rápido, y teniendo en cuenta la importancia de las fotos de cada plato, veo conveniente cargar las imágenes mediante un CDN. Esto también servirá para aprender de este tema. Al final de cuentas, estos proyectos de DevAsPros también deben servir de aprendizaje.

Otra forma es devolver versiones más pequeñas en la respuesta del JSON. No es necesario servir una imagen 4k en un celular.

Finalmente, dado lo poco cambiante que sería un menú, también se podría cachear la respuesta JSON usando la cabecera `Cache-Control` en el controlador.

## Cargar imágenes desde CDN

Para esto podemos usar CloudFront. Esta configuración se da mayoritariamente en la UI de AWS. Los siguientes tutoriales sirven de inicio para lograrla:

- [Rails, Assets, Active Storage and a Cloudfront CDN](https://headey.net/rails-assets-active-storage-and-a-cloudfront-cdn)
- [Using a CDN for Active Storage Uploads - Avo](https://avohq.io/blog/cdn-for-active-storage-uploads)

La clave aquí es usar el modo proxy de Active Storage como se menciona en la [documentación](https://guides.rubyonrails.org/v7.1/active_storage_overview.html#putting-a-cdn-in-front-of-active-storage).

## Servir variantes 

En lugar de devolver la imagen tal cual, aprovechar Active Storage para devolver versiones más pequeñas y que tenga menor tamaño.

Ver documentación: https://guides.rubyonrails.org/v7.1/active_storage_overview.html#transforming-images

> [!Tip]
> Para poder usar esta característica hay que instalar Vips o MiniMagick.

## Cachear respuesta JSON del controlador

Teniendo en cuenta lo poco que cambiaría cada plato del menú cachear la respuesta del JSON es una forma sencilla de mejorar la carga inicial. Para lograrlo se usa la cabecera `Cache-Control` seteado a 5 minutos (300 segundos).

Así es como gepeto sugiere hacerlo:
```ruby
class PublicMenusController < BasePublicMenuController
  def show
    restaurant = Restaurant.find_by!(slug: params[:slug])
    dishes = restaurant.dishes.with_attached_photo

    expires_in 5.minutes, public: true  # helper de Rails
    # o directamente:
    # response.headers["Cache-Control"] = "public, max-age=300"

    render inertia: "Menu",
           props: {
             dishes: DishBlueprint.render_as_hash(dishes, view: :short),
             name: restaurant.name,
             logo: restaurant.public_logo
           }
  end
end
```