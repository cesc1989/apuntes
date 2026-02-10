# Apuntes Ciclo 18

# Turbo Stream desde petición GET

Haciendo el botón "Ver temas cerrados" que estará en el detalle de Canal noté que quería usar un Turbo Stream de tal forma que al clicar el enlace la petición devolviese el stream.

Me ayudé en este tutorial -> https://www.colby.so/posts/infinite-scroll-with-turbo-streams-and-stimulus

## ¿Qué se necesita?

Una ruta y controlador común y corriente. El controlador tendrá una acción que responda una petición GET. Esta acción no tendrá `redirect_to` ni `render` normal.

```ruby
class ArchivedThemesController < ApplicationController
  def index
    channel = current_user.channels.find(params[:id])
    @archived_themes = channel.themes.archived
  end
end
```

El `link_to` debe usar el atributo `data` para poner dos opciones para Turbo Stream:
```ruby
<%= link_to(
      "Ver temas cerrados",
      "/channels/#{@channel.id}/archived/themes",
      class: "btn btn-secondary",
      data: { turbo_stream: "", turbo_method: :get }
    ) %>
```

La clave es `turbo_method: :get` para que haga una petición AJAX y no haga una navegación normal.

---

### ¿Por qué el enlace necesita data turbo_stream?

Si no se pone este atributo data tenemos este error al clicar el enlace:
```
Completed 406 Not Acceptable in 10ms (ActiveRecord: 0.3ms | Allocations: 3562)

ActionController::UnknownFormat (ArchivedThemesController#index is missing a template for this request format and variant.

request.formats: ["text/html"]
request.variant: []):
```

---

Finalmente, hay que tener un archivo `turbo_stream` que gestione la respuesta del controlador.

> Esto también se puede tener en el controlador pero lo hice así para seguir el tutorial. Además, en el controlador llevaría más cosas como un bloque `respond_to`.

El archivo `app/views/archived_themes/index.turbo_stream.erb` quedo así:
```ruby
<%= turbo_stream_action_tag(
  "append",
  target: "closed_themes",
  template: %(#{render partial: "themes/outside_of_channel_theme_link", collection: @archived_themes, as: :theme})
) %>
```

# Contando para verificar: `size` o `exists?` ?

> Este tema se puede encontrar en internets como "Presence Checking"

Escribí este método en el modelo `Channel`:
```ruby
class Channel < ApplicationRecord
  def archived_themes?
    themes.archived.size.positive?
  end
end
```

Para poder mostrar el enlace del botón "Ver temas cerrados" solo si el canal tiene temas cerrados. No tiene sentido que se muestre si no los tiene.

Sin embargo, me quedó una duda. ¿Hay mejor forma de hacerlo? Le pregunté a ChatGPT y me dijo que es mejor usar `exists?`. Quedó así luego de aplicar la sugerencia:
```ruby
  def archived_themes?
    themes.archived.exists?
  end
```

¿Hay alguna diferencia real? Probé en la consola de Rails y parece que sí. Aunque pequeña en este ejemplo.

```ruby
irb(main):002> ch1.themes.archived.size
  Theme Count (0.5ms)  SELECT COUNT(*) FROM "themes" WHERE "themes"."channel_id" = ? AND "themes"."closed" = ?  [["channel_id", 10], ["closed", 1]]
=> 8
irb(main):003> ch1.themes.archived.exists?
  Theme Exists? (0.2ms)  SELECT 1 AS one FROM "themes" WHERE "themes"."channel_id" = ? AND "themes"."closed" = ? LIMIT ?  [["channel_id", 10], ["closed", 1], ["LIMIT", 1]]
=> true
irb(main):004> ch1.themes.archived.size
  Theme Count (0.4ms)  SELECT COUNT(*) FROM "themes" WHERE "themes"."channel_id" = ? AND "themes"."closed" = ?  [["channel_id", 10], ["closed", 1]]
=> 8
irb(main):005> ch1.themes.archived.exists?
  Theme Exists? (0.2ms)  SELECT 1 AS one FROM "themes" WHERE "themes"."channel_id" = ? AND "themes"."closed" = ? LIMIT ?  [["channel_id", 10], ["closed", 1], ["LIMIT", 1]]
=> true
irb(main):006> ch1.themes.archived.size
  Theme Count (0.4ms)  SELECT COUNT(*) FROM "themes" WHERE "themes"."channel_id" = ? AND "themes"."closed" = ?  [["channel_id", 10], ["closed", 1]]
=> 8
irb(main):007> ch1.themes.archived.exists?
  Theme Exists? (0.2ms)  SELECT 1 AS one FROM "themes" WHERE "themes"."channel_id" = ? AND "themes"."closed" = ? LIMIT ?  [["channel_id", 10], ["closed", 1], ["LIMIT", 1]]
=> true
```

Nótese el tiempo de carga entre paréntesis. Así que parece que sí es más ventajoso usar `exists?` en lugar de contar todos los registros.

Este [artículo comenta](https://reinteractive.com/articles/Tutorial-Series-for-Experienced-Rails-Developers/activerecord-optimisation-utilising-exists-any-and-size) sobre esta situación y da otros ejemplos.