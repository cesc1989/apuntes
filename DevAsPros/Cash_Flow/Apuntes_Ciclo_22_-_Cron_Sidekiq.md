# Apuntes Ciclo 22 - Cron de Sidekiq

# Form edit con query params

Para poder cargar la lista de categorías en el formulario de nuevo gasto paso un query parameter al entrar al form:
```ruby
<%= link_to new_expenditure_path(parent_id: cat.id),
```

Y en el controlador uso el parámetro:
```ruby
def load_form_values
  @categories = Category.subcategories.where(parent_id: params[:parent_id])
end
```

Esto funciona bien al crear y editar con éxito. Pero cuando falla el form, al recargar la página la lista de categorías queda vacía.

¿Cómo le puedo pasar query params a la url del form?

## Crear nuevo recurso

Así se crea el form cuando es un nuevo recurso:
```ruby
<form data-turbo="false" action="/expenditures" accept-charset="UTF-8" method="post">

<input type="hidden" name="authenticity_token" value="CBN1Xm48c1FuxTTC1SiWoXsvmFLxsns4PY1Y4BRWfbBsN4IjDyJXBiqnH-r1mwY34svdvbuLPwS8QewhE8Q-Mg" autocomplete="off">
```

Atención con el atributo `action`:
```ruby
action="/expenditures"
```

y tiene método `post`. Esa ruta está definida como `POST /expenditures`.

## Actualizar recurso

En cambio, cuando se actualiza la ruta del formulario queda así:
```ruby
<form data-turbo="false" action="/expenditures/217" accept-charset="UTF-8" method="post">

<input type="hidden" name="_method" value="patch" autocomplete="off">

<input type="hidden" name="authenticity_token" value="EfC5KxDr2Qvqhj43cep7uCGinDlXZG6iqOUQMzIpdqk6kzYR7o3i3p74j7oVfQlVp1hFmUL6RokXAUHF36YdKA" autocomplete="off">
```

La ruta en `action` es `/expenditures/217` y usa un campo oculto para poder pasar el método `patch`.

## Enlaces sobre form_with

- form_with https://apidock.com/rails/ActionView/Helpers/FormHelper/form_with
- guías https://guides.rubyonrails.org/v7.0/form_helpers.html#dealing-with-model-objects
- Tuto en Medium https://medium.com/@eelan.tung/rails-forms-384cd22c65cc
- Otro tuto https://human-se.github.io/rails-demos-n-deets-2020/demo-resource-update/

# Campo `datetime_local_field` no aceptaba el input en Firefox Mobile

Desde que empecé a usar el campo `datetime_local_field` en el form, desde Firefox Mobile no se podía guardar gastos a menos que lo dejara tal cual y como cargaba el form.

Cuando carga el form se mostraba la fecha y hora con `DateTime.current`.

Probando con Ngrok, me di cuenta que la petición ni siquiera llegaba al servidor. Simplemente el botón guardar no hacía nada. El formulario tampoco mostraba errores y Firefox no indicaba nada.

Le pregunté a ChatGPT y la sugerencia fue forzar el formato esperado por el campo `datetime_local_field`. Lo hice así:
```ruby
<% expenditure_date = @expenditure.new_record? ? DateTime.current : @expenditure.spent_at %>

<%= form.datetime_local_field :spent_at, class: 'form-control', value: expenditure_date.strftime('%Y-%m-%dT%H:%M') %>
```

De esa forma pude probar y funcionaba de nuevo el form en Firefox Mobile. Todo era cuestión de forzar el formato. Según gepeto, es porque en navegadores Mobiles los estándares se aplican de manera más estricta.

# Asset  was not declared to be precompiled in production

Ya me había pasado antes este error en el Ciclo 17 -> [[Apuntes_Ciclo_17_-_Sidekiq_-_Cash_Flow#Problema de Assets Precompile con Capybara y System tests]]

```bash
Failure/Error: <%= javascript_importmap_tags %>

     ActionView::Template::Error:
       Asset `controllers/toast_controller.js` was not declared to be precompiled in production.
       Declare links to your assets in `app/assets/config/manifest.js`.

         //= link controllers/toast_controller.js

       and restart your server

   # --- Caused by: ---
	 # Sprockets::Rails::Helper::AssetNotPrecompiledError:
	 #   Asset `controllers/toast_controller.js` was not declared to be precompiled in production.
	 #   Declare links to your assets in `app/assets/config/manifest.js`.
	 #
	 #     //= link controllers/toast_controller.js
	 #
	 #   and restart your server
	 #   /Users/francisco/.gem/ruby/3.2.5/gems/sprockets-rails-3.4.2/lib/sprockets/rails/helper.rb:372:in `raise_unless_precompiled_asset'
```

Encontré una solución temporal en [este issue de 2020](https://github.com/rails/sprockets-rails/issues/458#issuecomment-618964357). Correr el comando:
```
bundle exec rake tmp:clear
```

Pero el error parece no haber tenido solución evidente.

# Revisando cron con Sidekiq Scheduler no ejecutando

Resulta que ya desde hace rato que en el servidor los crons de Sidekiq no se ejecutan. Más bien nunca. Al comienzo creía que era una mala configuración de Mailgun pero cuando probaba desde una consola de Rails, siempre recibía los correos. Decidí investigar.

Esto me llevó a revisar varios componentes:

- Sidekiq
- Sidekiq Schedulers
- Mailgun
- Reinicio del servidor por Unattended Upgrades

A continuación lo que revisé, algunos comandos para referencia y los descubrimientos.

## Tareas Recurrentes en Sidekiq Web

Están configuradas con Sidekiq Scheduler. Así se veía de primerazo:

![[001.crons.sidekiq.web.png]]

En la web, los crons decían que se iban a ejecutar pero nunca llegaban a ese momento. Entonces la configuración de Sidekiq Scheduler parecía estar en orden.

## Estado de nginx vs Sidekiq

Cuando revisaba el estado de los servicios en el servidor veía esto:

![[002.sidekiq.vs.nginx.status.png]]

Mira como el estado de activo de sidekiq decía "2min 42s ago". En cambio nginx decía "4 days ago".

Esto era una pista. ¿Por qué Sidekiq no estaba corriendo casi que a la par de Nginx? ¿Será que el reinicio cuando se aplican los upgrades afectaba a Sidekiq?

## Cambiando hora de los crons

Luego de muchos meses, volví para probar esto y nada que entendía el porque. Decidí probar cambiando el tiempo del cron del Chores Worker a cada minuto. De esa forma sí recibía los correos.

¿Por qué cuando estaba para correr en la madrugada no llegaban?

Así que le pedí ayuda a ChatGPT y ahí fuimos encontrando cosas.

## Hablando con ChatGPT

Me dijo que revisara los logs de sidekiq en el servidor.

Me dio este comando:
```bash
journalctl --user -u sidekiq.service --since "today"
```

El log fue extenso pero noté cierto patrón varias veces
```
Stopping sidekiq...
2025-02-28T02:02:51.074Z pid=87803 tid=1kkv INFO: Shutting down
2025-02-28T02:02:51.075Z pid=87803 tid=1kkv INFO: Terminating quiet threads
2025-02-28T02:02:51.143Z pid=87803 tid=1noz INFO: Scheduler exiting...
2025-02-28T02:02:51.644Z pid=87803 tid=1kkv INFO: Pausing to allow jobs to finish...
2025-02-28T02:02:53.147Z pid=87803 tid=1kkv INFO: Bye!
sidekiq.service: Succeeded.
Stopped sidekiq.
Started sidekiq.
```

Eso se veía muchas veces. Sidekiq se apagaba, luego se prendía. Y luego se apagaba. Así constantemente.

Tanto que siempre que entraba al servidor y ejecutaba el comando para revisar el estado de sidekiq veía que llevaba pocos segundos corriendo:
```bash
 systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-02-28 14:25:10 UTC; 10s ago
   Main PID: 115631 (bundle)
     CGroup: /user.slice/user-1000.slice/user@1000.service/sidekiq.service
             └─115631 sidekiq 6.5.12 app [0 of 1 busy]

Feb 28 14:25:10 localhost systemd[115624]: Started sidekiq.
Feb 28 14:25:13 localhost sidekiq[115631]: 2025-02-28T14:25:13.333Z pid=115631 tid=262j INFO: Booting Sidekiq 6.5.12 with Sidekiq::RedisConnection::RedisAdapter options {:url=>nil}
Feb 28 14:25:14 localhost sidekiq[115631]: 2025-02-28T14:25:14.364Z pid=115631 tid=262j INFO: Booted Rails 7.1.4 application in production environment
```

Esto nos llevó a descubrir algo: la variable de entorno de Redis no estaba seteada.

Este es el initializer:
```ruby
SIDEKIQ_REDIS_CONFIGURATION = {
  url: ENV["REDIS_URL"],
  ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE },
}

Sidekiq.configure_server do |config|
  config.redis = SIDEKIQ_REDIS_CONFIGURATION
end

Sidekiq.configure_client do |config|
  config.redis = SIDEKIQ_REDIS_CONFIGURATION
end
```

Cuando inspeccioné esa ENV devolvía nil. Y ese nil se puede apreciar en los logs:
```bash
INFO: Booting Sidekiq 6.5.12 with Sidekiq::RedisConnection::RedisAdapter options {:url=>nil}
```

Vaya fail de mí parte.

## Redis URL en el servidor

Antes de setear la URL de Redis, confirmé que estaba ejecutando:
```bash
systemctl status redis
● redis-server.service - Advanced key-value store
     Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-02-26 06:00:54 UTC; 2 days ago
       Docs: http://redis.io/documentation,
             man:redis-server(1)
   Main PID: 617 (redis-server)
      Tasks: 4 (limit: 1124)
     Memory: 7.9M
     CGroup: /system.slice/redis-server.service
             └─617 /usr/bin/redis-server 127.0.0.1:6379
```

Así que configuré la env con la correspondiente URL:
```bash
export REDIS_URL="redis://localhost:6379"
```

Luego, reinicié Sidekiq:
```bash
systemctl --user restart sidekiq.service
```

Y revisé de nuevo para encontrar que ahora sí carga la URL de Redis:
```bash
systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-02-28 15:38:56 UTC; 4s ago
   Main PID: 119522 (bundle)
     CGroup: /user.slice/user-1000.slice/user@1000.service/sidekiq.service
             └─119522 sidekiq 6.5.12 app [0 of 1 busy]

Feb 28 15:38:56 localhost systemd[119515]: Started sidekiq.
Feb 28 15:38:59 localhost sidekiq[119522]: 2025-02-28T15:38:59.398Z pid=119522 tid=292u INFO: Booting Sidekiq 6.5.12 with Sidekiq::RedisConnection::RedisAdapter options {:url=>"redis://localhost:6379"}
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.558Z pid=119522 tid=292u INFO: Booted Rails 7.1.4 application in production environment
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.559Z pid=119522 tid=292u INFO: Running in ruby 3.2.5 (2024-07-26 revision 31d0f1a2e7) [x86_64-linux]
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.559Z pid=119522 tid=292u INFO: See LICENSE and the LGPL-3.0 for licensing details.
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.559Z pid=119522 tid=292u INFO: Upgrade to Sidekiq Pro for more features and support: https://sidekiq.org
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.562Z pid=119522 tid=292u INFO: Loading Schedule
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.562Z pid=119522 tid=292u INFO: Scheduling recurring_expenditures {"class"=>"RecurringExpendituresWorker", "cron"=>"0 7 1 * *", "queue"=>"default"}
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.566Z pid=119522 tid=292u INFO: Scheduling daily_chores_check {"class"=>"DailyChoresCheckWorker", "cron"=>"0 6 * * *", "queue"=>"default"}
Feb 28 15:39:00 localhost sidekiq[119522]: 2025-02-28T15:39:00.568Z pid=119522 tid=292u INFO: Schedules Loaded
```

## Inspección de logs de servicio Sidekiq

Para revisar el log del demonio que corre sidekiq (recuerda que está configurado con systemd) uso este comando:

```bash
journalctl --user -u sidekiq.service -n 50 --no-pager
```

> Ver configuración de Sidekiq con system en [[Apuntes_Ciclo_17_-_Sidekiq_-_Cash_Flow]]

## Cada que entro al servidor, sidekiq lleva segundos corriendo

Sin embargo, cada que entro al servidor y reviso el estado de Sidekiq me dice que apenas lleva segundos corriendo.

```bash
systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-02-28 15:38:56 UTC; 4s ago
```

Mientras esté en el servidor conectado, el tiempo se mantiene como activo:
```bash
systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-02-28 15:38:56 UTC; 5min ago
```

Pero apenas salgo y vuelvo a entrar:
```bash
systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-02-28 15:45:26 UTC; 4s ago
```

Vuelve a reiniciar.

### systemd como usuario solo mientras el usuario está loggeado

El error de esto es porque configuré Sidekiq como demonio systemd pero solo se ejecuta mientras estoy loggeado.

Así lo dice el [artículo de donde saqué la configuración](https://dev.to/kevinluo201/start-sidekiq-6-as-daemon-in-production-environment-on-ubuntu-20-04-4m7b):
> User-wise systemd actually only allows logged-in user to execute the services.  
Therefore, all the services will be shut down while the last user session is ended.

Hay que hacer esto que me dijo Gpt:
```bash
loginctl enable-linger $(whoami)
```

lo cual también dice el autor del artículo.

Para revisar si un usuario de sistema (no root) tiene el linger activado puedo ejecutar este comando:
```bash
loginctl show-user $(whoami) | grep Linger
```

Si lo está, responde con:
```bash
Linger=yes
```

También se pueden listar todos los usuarios con el linger activado con:
```bash
loginctl list-users

 UID USER  
1000 ubuntu

1 users listed.
```

