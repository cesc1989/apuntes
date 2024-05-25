# Devise no cierra sesión [Coshi Notes]
Versiones:

    devise (4.9.3)
    rails (7.1.0)

Después de haber actualizado a Rails 7.1.0, haber tenido que fijar un par de gemas, y haber puesto la configuración para optimizar SQLite ocurría las siguientes cosas:

- En producción
    - en el celular, no se podía publicar ningún comentario porque redireccionaba a 404
    - en el pc, no podía cerrar sesión
- En local
    - no podía cerrar sesión


# En Development: Turbo y Devise

En el estado normal, el enlace de cerrar sesión está así:

    <%= link_to 'cerrar sessión', destroy_user_session_path, data: { turbo_method: :delete } %>

Pero cuando intento cerrar sesión la consola muestra:

    tSarted DELETE "/users/sign_out" for 127.0.0.1 at 2024-01-18 19:48:43 -0500
    Processing by Users::SessionsController#destroy as TURBO_STREAM
      User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 1], ["LIMIT", 1]]
    Redirected to http://localhost:3000/
    Completed 303 See Other in 5ms (ActiveRecord: 0.1ms | Allocations: 3082)
    
    
    Started GET "/" for 127.0.0.1 at 2024-01-18 19:48:43 -0500
    Processing by ChannelsController#index as TURBO_STREAM
      User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 1], ["LIMIT", 1]]
      Rendering layout layouts/application.html.erb
      Rendering channels/index.html.erb within layouts/application

Hacía la petición de manera correcta, empezaba la redirección y luego de llegar a la ruta principal, MANTENÍA LA SESIÓN.

    Started GET "/" for 127.0.0.1 at 2024-01-18 19:48:43 -0500
    Processing by ChannelsController#index as TURBO_STREAM

Probé de todo:

- usar un [button_to](https://github.com/heartcombo/devise/issues/5458#issuecomment-1151418373)
    - con method: :delete
- modificar el [controlador](https://github.com/heartcombo/devise/issues/5458#issuecomment-1022555755) de sesión de devise
- modificar el [inicializador](https://github.com/heartcombo/devise/issues/5458#issuecomment-1247433241) de devise

Y nada corregía el detalle hasta que cerré la sesión de Cashflow Dev. Luego volví a Coshi Notes Dev y todo volvió a la normalidad.

Si desactivaba turbo en el elemento padre, así salía la petición en la consola:

    Started DELETE "/users/sign_out" for 127.0.0.1 at 2024-01-18 19:40:41 -0500
    Processing by Users::SessionsController#destroy as HTML
      Parameters: {"authenticity_token"=>"[FILTERED]"}
      User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 1], ["LIMIT", 1]]
    Redirected to http://localhost:3000/
    Completed 303 See Other in 6ms (ActiveRecord: 0.1ms | Allocations: 4013)
    
    Started GET "/" for 127.0.0.1 at 2024-01-18 19:40:41 -0500
    Processing by ChannelsController#index as HTML
      User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 1], ["LIMIT", 1]]
      Rendering layout layouts/application.html.erb
      Rendering channels/index.html.erb within layouts/application
      Theme Load (1.2ms)  SELECT DISTINCT "themes".* FROM "themes" INNER J


## Cómo se ve todo el ciclo de peticiones cuando todo trabaja bien

Aquí se puede ver que la petición a ChannelsController está bien pero la respuesta es la diferencia.

En el caso de arriba del error, devuelve un 200. Aquí se puede ver que devolverá un 401.

    # Cerrar la sesión
    Started DELETE "/users/sign_out"
    Redirected to http://localhost:3001/
    Completed 303 See Other in 30ms
    
    # Intenter ver una página sin autorización. 401
    Started GET "/" for 127.0.0.1
    Processing by ChannelsController#index as TURBO_STREAM
    Completed 401 Unauthorized in 1ms 
    
    # Ir a página de inicio de sesión
    Started GET "/users/sign_in" for 127.0.0.1 at
    Processing by Users::SessionsController#new as TURBO_STREAM
    Completed 200 OK in 15ms



## Solución Temporal

Usar puertos diferentes para las aplicaciones locales que trabajo con Turbo.

Para Cashflow

    bundle exec rails server # puerto 3000

Para Coshi Notes

    bundle exec rails server -p 3001


# En Producción: force_ssl = true

Parece que en producción el asunto va por esa configuración que dejé activada.

En esta [pregunta](https://stackoverflow.com/questions/15676596/what-does-force-ssl-do-in-rails) se puede leer alguna que otra respuesta sobre qué hace esta configuración.

## ¿Qué hace activar force_ssl = true?

Documentación de [ActionDispatch::SSL](https://api.rubyonrails.org/classes/ActionDispatch/SSL.html).

Hace tres cosas activar esta configuración:

> **TLS redirect**: Permanently redirects `http://` requests to `https://` with the same URL host, path, etc.
> 
> **Secure cookies**: Sets the `secure` flag on cookies to tell browsers they must not be sent along with `http://` requests.
> 
> **HTTP Strict Transport Security (HSTS)**: Tells the browser to remember this site as TLS-only and automatically redirect non-TLS requests.


## Afectando el ingreso desde móvil

En el celular, ahora que desactivé eso no puedo iniciar sesión. Las peticiones se van a 404.

Incluso limpiando los datos solo del sitio como [explican acá](https://support.mozilla.org/en-US/kb/clear-cookies-and-website-data-single-domain-android).

