# ViewComponent - Apuntes

## Sensaciones Iniciales al Probar en Leisure Shelf Playground

Las sensaciones al usar View Component son muy buenas.

En realidad se siente mucho mejor escribir clases y templates apartes de la vista normal para manipular la salida a la pantalla.

No creé ni un solo partial para cosas como listar registros y se nota más simple al testear.
También, según artículos, es más óptimo usar Components que partials.

Sí queda ya es profundizar un poco más en temas como:

- Slots
- Sidecar assets

Pero lo básico, ya lo entendemos y tenemos una fuente para revisar en el futuro.

[Esto también](https://gitlab.com/cesc1989/leisure-shelf-playground/-/blob/master/app/components/activity_component.rb#L11) me recuerdan cuando en Macsa hice las "fachadas" aplicando el patrón Facade.

**No tenía que pensar en dónde ubicar métodos que iban a engordar el modelo o el controlador sino que están en su clase de Componente.**

Lo bueno de View Component es que no es solo para quitar partials con registros desde la base de datos sino que también para partes que son solo HTML.

Muy similar como en React lo de componentes funcionales y no funcionales / con estado.


## Usa submodulo `UI` para anidar los componentes y evitar ponerles el sufijo `Component`

Lo vi en este artículo: https://boringrails.com/articles/self-updating-components/

El autor dice:
> I love using `UI` module for components because it’s super clear this object is a view component and it lets you drop the “Component” suffix that people tend to use for every…single…component


Ejemplo:
```ruby
class UI::UserCard < ApplicationComponent
  def initialize(user:)
    @user = user
  end

  def id
    dom_id(@user, :user_card)
  end

  def broadcast_channel
    [@user, :user_card_refresh]
  end

  def broadcast_refresh!
    Turbo::StreamsChannel.broadcast_replace_to(
      broadcast_channel,
      target: id,
      renderable: self,
      layout: false
    )
  end
end
```

Esto me parece una forma interesante de organizar los componentes ya que se sale de eso de  Rails de dar sufijo al tipo de clase (controller, job, worker, service, etc).