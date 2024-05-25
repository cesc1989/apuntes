# Apuntes Ciclo 08 - Coshi Notes - Cache

# Hacer comentario enlazable

El helper link_to tiene la opción `anchor` para crear el ancla:

    <%= link_to channel_theme_path(channel, theme, anchor: dom_id(comment)) %>

Luego para copiar el enlace hay que hacerlo de esta forma:

    navigator.clipboard.writeText(this.commentLinkTarget.href).then(() => {})

donde `this.commentLinkTarget` es el object DOM y `href` es un atributo que devuelve toda la URL.


# Darle word-wrap a los contenedores de texto

Tenía este error:

![](https://paper-attachments.dropboxusercontent.com/s_80A21DFF3870D194C13D898912EFBA56723A3C8E851D480905C3ED302F83980F_1705453472531_imagen.png)


Eso pasa porque el texto de la URL es muy largo y no hay espaciado. Eso rompe el contenedor.

Para que el contenedor contenga el texto hay que usar la propiedad `word-wrap` como explican en [CSS Tricks](https://css-tricks.com/snippets/css/prevent-long-urls-from-breaking-out-of-container/).

    p {
        overflow-wrap: break-word;
        word-wrap: break-word;
        hyphens: auto;
    }


# Usando cache

Rails trae tres formas de hacer cache:

- Page cache
- Action cache
- Fragment cache

Según las guías, debo usar [Fragment Caching](https://guides.rubyonrails.org/caching_with_rails.html#fragment-caching).

Page Caching es para páginas que no son filtradas por `before_action`, es decir, páginas con autenticación no son cubiertas por este caso de uso.

> While this is super fast it can't be applied to every situation (such as pages that need authentication)

En desarrollo hay que activar/desactivar la cache:

    bin/rails dev:cache


## Cache en acción

Al cachear los canales en el sibebar, así muestra la consola el cacheo:

      Channel Load (0.1ms)  SELECT "channels".* FROM "channels" WHERE "channels"."user_id" = ? AND "channels"."archived" = ? ORDER BY "channels"."name" ASC  [["user_id", 1], ["archived", 0]]
      ↳ app/views/shared/_sidebar.html.erb:34
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/10-20240109025509108806 (0.1ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/11-20240109025509110412 (0.1ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/12-20240109025509111667 (0.1ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/13-20240109025509112892 (0.1ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/14-20240109025509113941 (0.2ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/15-20240109025509114943 (0.1ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/17-20240109025509118355 (0.0ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/19-20240109025509121246 (0.0ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/20-20240109025509122651 (0.0ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/21-20240109025509124567 (0.0ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/22-20240109025509126306 (0.0ms)
    Read fragment views/shared/_sidebar:b8545e0a6de45d0c3a052e833130e7aa/channels/23-20240109025509127673 (0.0ms)
      Rendered shared/_sidebar.html.erb (Duration: 3.0ms | Allocations: 2437) [cache hit]

En cambio, cuando lo desactivo:

      Channel Load (0.1ms)  SELECT "channels".* FROM "channels" WHERE "channels"."user_id" = ? AND "channels"."archived" = ? ORDER BY "channels"."name" ASC  [["user_id", 1], ["archived", 0]]
      ↳ app/views/shared/_sidebar.html.erb:34
      Rendered shared/_sidebar.html.erb (Duration: 2.5ms | Allocations: 2260)

Así configuré la cache en el sidebar:

    <% current_user.channels.visible.each do |channel| %>
      <% cache channel do %>
        <li class="nav-item">
          <%= link_to channel_path(channel),
              class: "nav-link d-flex align-items-center gap-2" do %>
                <i class="fas fa-hashtag"></i> <%= channel.name %>
          <% end %>
        </li>
      <% end %>
    <% end %>

Se envuelve el elemento en un bloque `cache do`.

En cambio, para un listado mediante partial es más sencillo:

    <%= render partial: 'themes/theme_link', collection: @themes, as: :theme, cached: true %>

Nota: por lo anterior, siempre será más conveniente cargar colecciones desde el controlador.

## Configurando la Cache en Producción

Se puede usar Redis, memchached, un directorio del servidor o la memoria del mismo.

Para el caso de Coshi Notes, como es una cosa que solo usaré yo y no tiene alto tráfico, puedo usar la opción de memoria:

    config.cache_store = :memory_store, { size: 64.megabytes }

Esto dicen las guías:

> This cache store is not appropriate for large application deployments. However, it can work well for small, low traffic sites with only a couple of server processes, as well as development and test environments.


# time_ago_in_words vs distance_of_time_in_words

Docs:

- [distance_of_time_in_words](https://apidock.com/rails/ActionView/Helpers/DateHelper/distance_of_time_in_words)
- [time_ago_in_words](https://apidock.com/rails/ActionView/Helpers/DateHelper/time_ago_in_words)

Estos métodos sirven para mostrar texto según la fecha de algo con respecto a otra fecha, por ejemplo:

- hace 5 horas
- poco más de 1 año

etc.

Para mostrar hace cuanto se posteó un comentario usé `time_ago_in_words` así:

    <%= time_ago_in_words(theme.last_comment.created_at) %>

El helper `distance_of_time_in_words` me costó usarlo bien por los ejemplos:

    distance_of_time_in_words(from_time, from_time + 50.minutes)

pero la realidad es que `time_ago_in_words` usa el mismo helper con el segundo parámetro fijo. Así lo vemos en el código fuente:

    def time_ago_in_words(from_time, options = {})
      distance_of_time_in_words(from_time, Time.now, options)
    end

