# Apuntes Ciclo 03 - Coshi Notes - Turbo Frame Editar

# Editar comentario mediante Turbo Frame

Quería lograr que el formulario se abriera en el mismo lugar donde está el comentario, editarlo y mostrar el nuevo contenido.

Lo logré con Turbo Frame pero me costó algo por las muchas cosas envueltas.

Veamos.

## Cosas Importantes
- Hay que tener en cuenta la carga de HTML normal y la mediante Turbo al hacer la petición
- Todo lo que devuelva el Turbo Stream en la respuesta del controlador, debería hacerlo el callback que haya en el modelo si usa el macro `broadcast_x_to`.
- Hay que envolver el elemento HTML a editar y el form en la vista `edit.html.erb` con un `turbo_frame_tag`.
    - Ambos deben tener el mismo ID generado con `dom_id(object)`


## Lo que se envía para el HTML mediante Turbo debe estar también en la vista en modo normal

Terminé necesitando la variable de instancia para `@channel` y para `@theme` ya que necesitaba armar la ruta para cancelar la edición.

    <%= link_to 'Cancelar', channel_theme_path(channel, theme), class: 'btn btn-danger' %>

De esa forma se navega a la misma página pero Turbo hace que no se refresque y todo parece normal.

Debido a eso tocó mandar esas dos variables en locals en muchas partes.

    # app/controllers/comments_controller.rb
    locals: { comment: @comment, channel: @comment.channel, theme: @comment.theme }
    
    # app/models/comment.rb
    broadcast_replace_to 'comments', partial: 'comments/comment', locals: { channel: channel, theme: theme }
    
    # app/views/comments/edit.html.erb
    <%= render partial: 'form', locals: { comment: @comment, channel: @channel, theme: @theme } %>
    
    # app/views/themes/show.html.erb
    <%= render partial: 'comments/form', locals: { comment: @comment, channel: @channel, theme: @theme } %>

**¿Se puede hacer de una mejor manera?** Para poder quitar todos esos `locals: {}`.


## Envolver elemento HTML y vista editar en turbo_frame_tag

Deben quedar así.

Elemento HTML:

    # app/views/comments/_comment.html.erb
    
    <%= turbo_frame_tag dom_id(comment) do %>
      <div class="row">
        <!-- (...) -->
        <div class="col-12 tiptap_content">
          <%= (comment.plain_content || "").html_safe %>
        </div>
      </div>
    <% end %>

Vista editar:

    <%= turbo_frame_tag dom_id(@comment) do %>
      <%= render partial: 'form', locals: { comment: @comment, channel: @channel, theme: @theme } %>
    <% end %>


## Turbo Stream igual en controlador y modelo

Sino tenía el controlador:

    format.turbo_stream do
      render(
        turbo_stream: turbo_stream.append(
          @comment,
          partial: 'comments/comment',
          locals: { comment: @comment, channel: @comment.channel, theme: @comment.theme }
        )
      )
    end

Y el modelo:

    after_update_commit do
      broadcast_replace_to 'comments', partial: 'comments/comment', locals: { channel: channel, theme: theme }
    end

Iguales con respecto a la línea:

    locals: { comment: @comment, channel: @comment.channel, theme: @comment.theme }

No terminaba de funcionar la transmisión del elemento HTML.

