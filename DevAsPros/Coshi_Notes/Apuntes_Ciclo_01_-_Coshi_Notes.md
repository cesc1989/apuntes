# Apuntes Ciclo 01 - Coshi Notes

# Turbo Stream: Lo necesario

Cómo hacer que cambios guardados sean enviados y reflejados en TurboStreams. Así lo hice para guardar un Comentario en la vista de Tema.

## En la vista

En este caso es en la vista show de Themes

    <%= turbo_frame_tag 'comments' do %>
      <%= render partial: 'comments/comment', collection: @theme.comments, as: :comment %>
    <% end %>
    
    <%= turbo_stream_from 'comments' %>

Sin el helper `turbo_stream_from` no funciona. Sin envolver el listado de “comments” en `turbo_frame_tag` no funciona.

**turbo_stream_from**
El helper `turbo_stream_from` es el que establece la conexión websocket.

Genera algo como esto:

    <turbo-cable-stream-source channel="Turbo::StreamsChannel" signed-stream-name="ImNvbW1lbnRzIg==--e3b5ab320816f9869628fd667e000208a2e88568a30b68949bdc5c95e8fa9434" connected=""></turbo-cable-stream-source>

Y se puede ver la conexión en el inspector de red:

![](https://paper-attachments.dropboxusercontent.com/s_513C52A4BEC82A34484950D1D7ACD893A1C4AB27852B2A87245C2DC09823900E_1704488217793_imagen.png)



## En el modelo

Debe estar el macro `after_create_commit` para que haga la transmisión.

    class Comment < ApplicationRecord
      after_create_commit { broadcast_append_to 'comments' }
    end


## En el controlador

Descubrí que no es necesario que esté el response `turbo_stream` cuando el modelo está configurado para hacer el broadcast.

Es más, tuve problemas la dejar ambos: modelo y controlador.

    class CommentsController < ApplicationController
      def create
        respond_to do |format|
          if @comment.save
            format.html {}
            # funciona el streaming con o sin esta línea
            # format.turbo_stream {}
          end
        end
      end
    end

Si se deja la respuesta turbo_stream vacía, la respuesta de la petición será `head: :no_content`:

    [ActionCable] Broadcasting to comments: "<turbo-stream action=\"append\" target=\"comments\"><template><turbo-frame id=\"comment_191\">\n  <div class=\"row\">\n    <div class=\"col-12\">\n      <small>Francisco Quintero</small>\n      <small class=\"created_at\">05/01/2024</small>\n\n      <a class=\"edit_comment\" href=\"/comments/191...
    No template found for CommentsController#create, rendering head :no_content
    Completed 204 No Content in 30ms (ActiveRecord: 2.4ms | Allocations: 7623)


## En el formulario

Hay que activar Turbo para el form

    <%= form_with(model: @comment, local: true, data: { turbo: true }) do |f| %>


Enlaces

- Tutorial para formularios con Turbo y Hotwire → https://www.hotrails.dev/turbo-rails/turbo-frames-and-turbo-streams


# Correcto funcionamiento de Turbo

Para que funcione bien Turbo hay que agregar Popper y bootstrap en

- config/initializers/assets.rb
- app/javascript/application.js
- config/importmap.rb


# Desplazar barra de navegación hacía abajo

Se necesita que un elemento esté en el fondo de la página y luego se busca mediante selectores de elementos. Una vez elegido, se usa la función `scrollIntoView()`

Así quedó el controlador Stimulus

    import { Controller } from "@hotwired/stimulus"
    
    export default class extends Controller {
      static targets = ["view"]
    
      connect() {
        const view = this.viewTarget.dataset.viewName;
    
        if (view == "themes_show") {
          const commentBox = document.querySelector("#new_comment");
          commentBox.scrollIntoView({ behavior: "smooth" });
        }
      }
    }
    


Enlaces:

- [Stack Overflow](https://stackoverflow.com/questions/40903462/how-to-keep-a-scrollbar-always-bottom/75296945#75296945)
- [MDN scrollIntoView](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView)

