# Lecciones y Aprendizajes #1

- Documentación del helper `link_to` [https://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to](https://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to)
- Cómo incluir imágenes de los assets en hoja de estilo sass: [https://guides.rubyonrails.org/asset_pipeline.html#css-and-sass](https://guides.rubyonrails.org/asset_pipeline.html#css-and-sass)
- `date_select` helper https://apidock.com/rails/ActionView/Helpers/DateHelper/date_select
- Autoreloading from Vagrant: [https://stackoverflow.com/questions/35959490/rails-server-doesnt-see-code-changes-and-reload-files#36616931](https://stackoverflow.com/questions/35959490/rails-server-doesnt-see-code-changes-and-reload-files#36616931)


## Error de la gema autoprefixer (incluida en bootstrap)
    Autoprefixer doesn’t support Node v0.10.46. Update it.

Solución: instalar una versión nueva de node y también remover versiones viejas porque la gema `exec-js` parece que toma una de las dos. Ver:

- https://github.com/ai/autoprefixer-rails/blob/master/lib/autoprefixer-rails/processor.rb#L159
- También comenté en el issue https://github.com/ai/autoprefixer-rails/issues/140


## Configuración del Procfile
    This address is restricted
    
    This address uses a network port which is normally used for purposes other than Web browsing. Firefox has canceled the request for your protection.

Resulta que el puerto 6000 hace parte de una lista de puertos bloqueados por motivos de seguridad.

- Lista para Chrome: https://superuser.com/a/188070/372807
- Lista para Firefox: https://www-archive.mozilla.org/projects/netlib/PortBanning.html#portlist


## Gema Simple Form
- Gema: https://github.com/plataformatec/simple_form
- Ejemplo que usa bootstrap 4: https://github.com/rafaelfranca/simple_form-bootstrap/blob/master/Gemfile
- Bootstrap gem: https://github.com/twbs/bootstrap-rubygem


## Configuración de Vagrant

Como siempre algún error sale:

    fatal: [avanzza]: FAILED! => {"changed": false, "msg": "Failed to validate the SSL certificate for deb.nodesource.com:443. Make sure your managed systems have a valid CA certificate installed. If the website serving the url uses SNI you need python >= 2.7.9 on your managed machine  (the python executable used (/usr/bin/python) is version: 2.7.6 (default, Nov 23 2017, 15:49:48) [GCC 4.8.4]) or you can install the `urllib3`, `pyOpenSSL`, `ndg-httpsclient`, and `pyasn1` python modules to perform SNI verification in python >= 2.6. You can use validate_certs=False if you do not need to confirm the servers identity but this is unsafe and not recommended. Paths checked for this platform: /etc/ssl/certs, /etc/pki/ca-trust/extracted/pem, /etc/pki/tls/certs, /usr/share/ca-certificates/cacert.org, /etc/ansible. The exception msg was: [Errno 1] _ssl.c:510: error:14077410:SSL routines:SSL23_GET_SERVER_HELLO:sslv3 alert handshake failure."}

Posibles soluciones al error del comentario de abajo:

- https://apassionatechie.wordpress.com/2017/12/24/ansible-failed-to-validate-the-ssl-certificate/
- Lo que en realidad ayudó está acá: https://groups.google.com/forum/#!msg/ansible-project/p4dQ0c25bpM/qSsI4JQqBAAJ

Cómo también indiqué acá: https://github.com/nodesource/ansible-nodejs-role/issues/33

## Error undefined method `stringify_keys' for

El error:

    undefined method `stringify_keys' for "/owners/1/vehicles":String excluded from capture: DSN not set
    14:12:56 rails.1 |   
    14:12:56 rails.1 | ActionView::Template::Error (undefined method `stringify_keys' for "/owners/1/vehicles":String):
    14:12:56 rails.1 |     18:               | Administrativa
    14:12:56 rails.1 |     19:           ul.collapse aria-expanded="false"
    14:12:56 rails.1 |     20:             li
    14:12:56 rails.1 |     21:               = link_to 'Vehículos', owner_vehicles_path(current_owner)
    14:12:56 rails.1 |     22:                 i.fa.fa-car
    14:12:56 rails.1 |     23:             li
    14:12:56 rails.1 |     24:               = link_to 'Conductores', owner_drivers_path(current_owner)

ocurría porque en helper `[link_to](https://apidock.com/rails/v4.2.7/ActionView/Helpers/UrlHelper/link_to)` en su forma en bloque recibe como primer argumento una ruta y no una cadena. Solución:

    = link_to owner_vehicles_path(current_owner)
                    i.fa.fa-car
                    |  Vehículos

Lo vi en Stack Overflow: https://stackoverflow.com/questions/35455132/nested-icons-and-rails-link-to-or-button-to-are-not-working-with-slim


## Configuración ideal para íconos de Font Awesome

Así https://gist.github.com/cesc1989/18ecec1fc6ef1c16a9dd#the-recommended-way


## Configurando devise para más de un modelo
- Con respecto a Vistas: https://github.com/plataformatec/devise#configuring-views
- Con respecto a Controladores: https://github.com/plataformatec/devise#configuring-controllers
- Con respecto a Modelos: https://github.com/plataformatec/devise#configuring-multiple-models
> Keep in mind that those models will have completely different routes. They **do not and cannot** share the same controller for sign in, sign out and so on. In case you want to have different roles sharing the same actions, we recommend that you use a role-based approach, by either providing a role column or using a dedicated gem for authorization.



## Usando Cloudinary con ActiveStorage
- Cloudinary y Rails [https://cloudinary.com/documentation/rails_integration](https://cloudinary.com/documentation/rails_integration)
- Gema activestorage-cloudinary https://github.com/0sc/activestorage-cloudinary-service
- Service Cloudinary para Active Storage: [https://support.cloudinary.com/hc/en-us/community/posts/115003165791-Rails-5-2-ActiveStorage-support-for-Cloudinary-Service](https://support.cloudinary.com/hc/en-us/community/posts/115003165791-Rails-5-2-ActiveStorage-support-for-Cloudinary-Service)
- Cloudinary integration with Rails 4 [https://devcenter.heroku.com/articles/cloudinary#using-with-ruby-on-rails](https://devcenter.heroku.com/articles/cloudinary#using-with-ruby-on-rails)


