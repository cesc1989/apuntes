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
