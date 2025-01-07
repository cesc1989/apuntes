# RFC 001: Comentarios como Favoritos

En Coshi Notes se marcan los comentarios como favoritos cambiando una bandera booleana.

```ruby
<%= button_to comment_path(comment),
	class: "dropdown-item",
	params: { comment: { favorited: !comment.favorited }},
	method: :put,
	title: "Marcar como favorito" do %>
		<i class="fas fa-bookmark"></i> <%= comment.favorited ? "Desfavoritear" : "Favoritear" %>
<% end %>
```

Los favoritos se muestran en una página aparte con el estilo de Tema en vista de Canal

![[coshinotes.favs.png]]

Así se cargan:
```ruby
class FavoritesController < ApplicationController
  def index
    @comments = current_user.favorites.includes(theme: :channel)
  end
end

# modelo User
def favorites
  comments.where(favorited: true).order(updated_at: :desc)
end
```

La forma actual funciona para lo que necesito pero siento que todo gira entorno al tema. El título principal del bloque de favorito es el nombre del Tema y luego el texto del comentario le sigue.

Está de esa forma porque normalmente el texto del comentario es largo.

## Cambiar la UI

Había pensado en usar una asociación polimorfíca para tratar de darle más prominencia al comentario pero no es necesario. ==En cambio, puedo cambiar la UI de cómo se muestra el favorito para lograr ese efecto==.

Hay que darle la vuelta a este bloque HTML:
```html
<div class="row mb-3 mb-md-0">
	<div class="col-12 col-md-10">
		<%= link_to channel_theme_path(comment.channel, theme, anchor: dom_id(comment)), class: "theme-link" do %>
			<h5 class="<%= closed_channel_color(theme.closed) %>">
				<% if theme.closed %>
					<i class="fas fa-comment-slash fs-6"></i>
				<% end %>
	
				En <%= theme.name %>
	
			 <small>
				 (<%= theme.channel_name %>)
			 </small>
			</h5>
	
			<p>
				<%= strip_tags(comment.plain_content.truncate(200)) %>
			</p>
		<% end %>
	</div>
</div>
```

Y darle un vuelco para que el texto del comentario sea lo que más se vea y el nombre del canal quede relegado al 2do plano.

Podría optar por las [tarjetas de Boostrap](https://getbootstrap.com/docs/5.3/components/card/#about). Estas ya las use en Cash Flow para mostrar las Tareas Recurrentes. Creo que tienen un buen formato donde el `card-body` es el lugar ideal para el texto del comentario y el `card-header` o `card-footer` pueden darle ese 2do plano que quiero para el nombre del tema.