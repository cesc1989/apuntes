# Apuntes Ciclo 17 - Sidekiq - Cash Flow

# Correr Sidekiq en servidor Ubuntu

Tenía que instalarlo y había que ejecutarlo como demonio de servicio. Encontré un tutorial donde explican y me sirvió de punto de partida.

Recursos:

- [Run Sidekiq 6 as daemon in Production environment on Ubuntu 20.04](https://dev.to/kevinluo201/start-sidekiq-6-as-daemon-in-production-environment-on-ubuntu-20-04-4m7b)
- [Understanding Sidekiq's systemd Service Unit File](https://joelngwt.github.io/2023/12/02/understanding-sidekiq-systemd-service-unit-file.html)
- Configuración de variables de entorno en systemd [https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#Environment](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#Environment)

Así queda la configuración final para dejarlo como servicio de Systemd.

    # /home/ubuntu/.config/systemd/user/sidekiq.service
    
    [Unit]
    Description=sidekiq
    After=syslog.target network.target
    
    [Service]
    Type=simple
    
    WorkingDirectory=/home/ubuntu/cashflow/app
    
    Environment=MALLOC_ARENA_MAX=2
    
    EnvironmentFile=/home/ubuntu/.sidekiq_envs
    ExecStart=/opt/rubies/ruby-3.1.0/bin/bundle exec sidekiq -e production
    
    ExecReload=/usr/bin/kill -TSTP $MAINPID
    
    RestartSec=1
    Restart=on-failure
    
    StandardOutput=syslog
    StandardError=syslog
    
    SyslogIdentifier=sidekiq
    
    [Install]
    WantedBy=default.target

Esta es la configuración como servicio de usuario. Es decir, se puede ejecutar sin sudo.

Para este caso el archivo debe ubicarse en la carpeta `~.config/systemd/user/`.

## Activación

Una vez el archivo copiado a la carpeta esperada, se ejecutan estos comandos para activar el servicio:

    systemctl --user daemon-reload
    systemctl --user enable sidekiq.service

Y con este otro se verifica el estado del mismo. Así se puede ver si está ejecutando o si hay errores de configuración o ejecución.

    systemctl --user status sidekiq.service

Ejemplo:

    systemctl --user status sidekiq
            > ● sidekiq.service - sidekiq
            > Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
            > Active: inactive (dead)
            > Jan 31 01:06:37 localhost systemd[3611161]: /home/ubuntu/.config/systemd/user/sidekiq.service:15: Executable "bundle" not found in path>
            > Jan 31 01:06:37 localhost systemd[3611161]: sidekiq.service: Unit configuration has fatal error, unit will not be started.


## Comandos de uso

Estos son todos los comandos disponibles:

    systemctl --user stop sidekiq.service
    systemctl --user start sidekiq.service
    systemctl --user restart sidekiq.service
    systemctl --user status sidekiq.service


## Variables de Entorno

El comando de la directiva `ExecStart` necesita las variables de entorno normales de Rails.

Hay dos formas de pasarlas.

En línea

    ExecStart=sh -c 'export SECRET_KEY_BASE=supermegahypersecretsecretosuperyeahmaracueya && /opt/rubies/ruby-3.1.0/bin/bundle exec sidekiq -e production'

O mediante directivas:

    EnvironmentFile=/home/ubuntu/.sidekiq_envs
    ExecStart=/opt/rubies/ruby-3.1.0/bin/bundle exec sidekiq -e production

Para esto preferí la forma mediante directivas porque es más limpia.


## Type=simple o Type=notify

Finalmente, tocó usar Type=simple porque el comando para arrancar Sidekiq no forkea un proceso. Es decir, se mantiene en el mismo proceso hasta que se termine mediante CTRL + C.

Y con todo eso fue que Sidekiq quedó corriendo como servicio de sistema en Systemd.


# Problema de Assets Precompile con Capybara y System tests

En los tests de systema, en específico para la página de inicio de sesión, tenía este problema:

    Failure/Error: <%= stylesheet_link_tag 'application', media: 'all' %>
    
         ActionView::Template::Error:
           Asset `application.css` was not declared to be precompiled in production.
           Declare links to your assets in `app/assets/config/manifest.js`.
    
             //= link application.css
    
           and restart your server
         # /Users/francisco/.gem/ruby/3.1.0/gems/sprockets-rails-3.4.2/lib/sprockets/rails/helper.rb:372:in `raise_unless_precompiled_asset'
         # /Users/francisco/.gem/ruby/3.1.0/gems/sprockets-rails-3.4.2/lib/sprockets/rails/helper.rb:338:in `digest_path'
         # /Users/francisco/.gem/ruby/3.1.0/gems/sprockets-rails-3.4.2/lib/sprockets/rails/helper.rb:326:in `asset_path'
    
         # ./spec/system/sessions/user_sign_in_spec.rb:27:in `block (3 levels) in <top (required)>'
         # ./spec/support/database_cleaner.rb:9:in `block (3 levels) in <top (required)>'
         # /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/strategy.rb:30:in `cleaning'
         # /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/cleaners.rb:34:in `block (2 levels) in cleaning'
         # /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/cleaners.rb:35:in `cleaning'
         # ./spec/support/database_cleaner.rb:8:in `block (2 levels) in <top (required)>'
         # ------------------
         # --- Caused by: ---
         # Sprockets::Rails::Helper::AssetNotPrecompiledError:
         #   Asset `application.css` was not declared to be precompiled in production.
         #   Declare links to your assets in `app/assets/config/manifest.js`.
         #
         #     //= link application.css
         #
         #   and restart your server

El problema era que tenía en uso dos

    <%= stylesheet_link_tag 'name', media: 'all' %>

en el layout de login. El que venía en el partial head y el de login. Mi sospecha es que el de “login” no se pasaba por el precompilado y por eso explotaba con ese error.

Resolví incluyendo el archivo “login.css” en “application.scss” y solo usar un stylesheet_link_tag.


> Nota: esto también me tocó hacerlo en Coshi Notes.



# Desfase de fechas y Date.today con TimeZone

El resumen es que siempre que configuré timezone en una app rails debo usar `Date.current` o los métodos de `Time` o `DateTime` que soportan zona horaria.

[Docs de Date.current](https://api.rubyonrails.org/classes/Date.html#method-c-current)

> Returns `Time.zone`.today when `Time.zone` or `config.time_zone` are set, otherwise just returns Date.today.

El artículo que me abrió los ojos http://danilenko.org/2012/7/6/rails_timezones/

## Resumen

**Get current time**
```ruby
# Correct
Time.zone.now

# Acceptable
Time.now.in_time_zone
DateTime.now.in_time_zone

# Wrong
Time.now
DateTime.now
```

**Get Day (Today, Yesterday, etc.)**
```ruby
# Correct
Time.zone.today
Time.zone.today - 1.day

# Aceptable
Date.current
Date.yesterday

# Wrong
Date.today
```

**Build time**
```ruby
# Correct
Time.zone.local(2012, 6, 10, 12, 00)

# Wrong
Time.new(2012, 6, 10, 12, 00)
DateTime.new(2012, 6, 10, 12, 00)
```

**Time From Timestamp**
```ruby
# Correct
Time.zone.at(timestamp)

# Acceptable
Time.at(timestamp).in_time_zone

# Wrong
Time.at(timestamp)
```


