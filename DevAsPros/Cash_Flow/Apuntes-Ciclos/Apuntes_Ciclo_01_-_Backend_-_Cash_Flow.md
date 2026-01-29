# Apuntes Ciclo 01 - Backend - Cash Flow
Aquí hay cosas para tener en cuenta en Puntapie.

# Desactivar Turbo para link_to

Como traje la plantilla Puntapie ya esto estaba cubierto pero había problema para que Rails cogiera un link de esta forma:

    <%= link_to "Logout", destroy_user_session_path, method: :delete %>

Y no funciona eso por Turbo.

¿Solución?

    <%= link_to "Logout", destroy_user_session_path, data: { turbo_method: :delete } %>

Visto aquí [https://dev.to/spaquet/rails-7-devise-log-out-d1n](https://dev.to/spaquet/rails-7-devise-log-out-d1n) y aquí [https://chimkan.com/rails-7-and-devise-issue-with-sign-out/](https://chimkan.com/rails-7-and-devise-issue-with-sign-out/)


## Actualización 19 de Abril 2024

En una app nueva hecha desde Puntapie tuve este problema y poner el atributo data no sirvió.

Lo que sí sirvió fue esto:

    button_to "Log out", destroy_user_session_path, method: :delete

Aunque el botón se veía horrible y no quería gastarle energías a cambiar los estilos. Entonces me funcionó cambiar este setting para seguir usando el enlace normal.

    # config/devise.rb
    # config.sign_out_via = :delete
    config.sign_out_via = :get

Fuente https://github.com/heartcombo/devise/issues/4570#issuecomment-780481753


# Arreglar "waiting for runner" en GH Action

La solución es usar `ubuntu-latests` según [https://stackoverflow.com/questions/70959954/error-waiting-for-a-runner-to-pick-up-this-job-using-github-actions](https://stackoverflow.com/questions/70959954/error-waiting-for-a-runner-to-pick-up-this-job-using-github-actions)


# Corregir error de plataforma en Gemfile.lock

Como hice bundle install desde mac, al subir al servidor Linux hubo error de plataforma no soportada.

Se arregla con

    bundle lock --add-platform x86_64-linux


# Configurar Bootstrap con Importmaps

Encontré en este artículo la forma de configurar Bootstrap con Importmaps. Aprovecha que se tiene la gema instalada https://dev.to/coorasse/rails-7-bootstrap-5-and-importmaps-without-nodejs-4g8

Y Custom layout en Devise [https://github.com/heartcombo/devise/wiki/How-To:-Create-custom-layouts](https://github.com/heartcombo/devise/wiki/How-To:-Create-custom-layouts)

