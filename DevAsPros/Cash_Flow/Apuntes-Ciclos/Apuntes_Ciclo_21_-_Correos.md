# Apuntes Ciclo 21 - Envío de Correos

# Correos no salen desde el servidor

Pasaba este error:
```
Delivered mail 671664e31ebd0_29dda4ab0200cd@localhost.mail (2.9ms)
/home/ubuntu/cashflow/deployments/api-gems/bundle/ruby/3.1.0/gems/net-smtp-0.5.0/lib/net/smtp/authenticator.rb:20:in `check_args': SMTP-AUTH requested but missing user name (ArgumentError)
```

Y era porque tenía mal configurada la variable de entorno para el user name.

Antes:

```
export tMAILGUN_USERNAME=
```

Después:

```
export MAILGUN_USERNAME=
```

Ahora el error es otro:

```
Delivered mail 6716653cba6a7_29eca4ab06316b@localhost.mail (168.6ms)

/home/ubuntu/cashflow/deployments/api-gems/bundle/ruby/3.1.0/gems/net-smtp-0.5.0/lib/net/smtp.rb:1042:in `check_continue': could not get 3xx (421: 421 Domain sandbox947896f1956b403197c80061aa04c9e4.mailgun.org is not allowed to send: Sandbox subdomains are for test purposes only. Please add your own domain or add the address to authorized recipients in Account Settings. (Net::SMTPUnknownError)

)
```

# Configuración subdominio en Namecheap para envío de correo desde Mailgun

La clave de la configuración es que cuando se creen los registros TXT no hay que poner el nombre del dominio en el campo Host.

Esto no:
```
pic._domainkey.mg.devaspros.com
```

Eso sí:
```
pic._domainkey.mg
```

Otro ejemplo.

Así no:
```
mg.devaspros.com
```

Así sí:
```
mg
```

Para el registro CNAME la dinámica es similar para el campo Host.

Esto no:
```
email.mg.devaspros.com
```

Eso sí:
```
email
```

# Actualización a Rails 7.1.4

Todo bien excepto al correr las pruebas hay errores y varios mensajes de DEPRECATION.

## DeprecatedConstantAccessor.deprecate_constant without a deprecator is deprecated

```
DEPRECATION WARNING: DeprecatedConstantAccessor.deprecate_constant without a deprecator is deprecated (called from <top (required)> at /Users/francisco/projects/devaspros-projects/cashflow/config/application.rb:19)

DEPRECATION WARNING: DeprecatedConstantAccessor.deprecate_constant without a deprecator is deprecated (called from <top (required)> at /Users/francisco/projects/devaspros-projects/cashflow/config/application.rb:19)
```

==Este se resuelve al actualizar Devise a la versión 4.9.3== -> https://github.com/heartcombo/devise/blob/v4.9.3/CHANGELOG.md#493---2023-10-11

## TestFixtures.fixture_path= is deprecated

Otro mensaje de depreciación es este:
```
DEPRECATION WARNING: TestFixtures.fixture_path= is deprecated and will be removed in Rails 7.2. Use .fixture_paths= instead. (called from <top (required)> at /Users/francisco/projects/devaspros-projects/cashflow/spec/components/base_income_component_spec.rb:7)

DEPRECATION WARNING: TestFixtures.fixture_path= is deprecated and will be removed in Rails 7.2. Use .fixture_paths= instead. (called from <top (required)> at /Users/francisco/projects/devaspros-projects/cashflow/spec/components/total_month_expenditure_component_spec.rb:7)
```

Este se resuelve al actualizar rspec-rails a la versión 6.0.2 -> https://github.com/rspec/rspec-rails/pull/2664#issuecomment-1534253592

# Fechas y Rangos

Es mejor usar `Time.zone.now` a `DateTime.current` para rangos de fecha. Usando `Time` se tiene en cuenta el timestamp de los registros y así se incluye todo.

Mucho más práctico usar:
```ruby
date.all_day
date.all_week
date.all_month
date.all_quarter
date.all_year
```

A crear un rango manualmente:
```ruby
[Date.current.beginning_of_month..Date.current.end_of_month]
```

Visto en https://rails.rubystyle.guide/#date-time-range

# Arreglo correcto para gemas por defecto de Ruby

Etiquetas: #passenger

Las gemas base64 y stringio tienen versiones por defecto en Ruby. Si se instala una versión diferente passenger explota y el log dice algo sobre que la versión activada tiene conflicto con la versión de Ruby.

Entonces toca forzar en el Gemfile o ver qué se hace.

La solución correcta, cuando se usa Passenger como app server es poner esto en el archivo nginx.conf:
```
passenger_preload_bundler on;
```

Según la [documentación](https://www.phusionpassenger.com/docs/references/config_reference/nginx/#passenger_preload_bundler):
> If this option is turned on, Ruby will be instructed to load the bundler gem before loading your application. This can help with gem version conflicts due to order-of require issues.

Lo encontré en [este hilo](https://www.reddit.com/r/rails/comments/18105z2/ruby_on_rails_phusion_passenger_error/). El mismo Chris Oliver comenta la solución.

# Correos no salían desde Sidekiq

> [!note]
> Faltaba configurar las envs de Mailgun para que las leyera el demonio de Sidekiq.

El error que vi al ver los logs:
```
sudo tail -n50 /var/log/syslog
```

El error del puerto 587:
```
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.497Z pid=497708 tid=b4ts class=DailyChoresCheckWorker jid=4e5ca77dfcced746f0de6ec0 INFO: start
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.540Z pid=497708 tid=b4ts class=DailyChoresCheckWorker jid=4e5ca77dfcced746f0de6ec0 INFO:   Rendered layout layouts/mailer.html.erb (Duration: 1.6ms | Allocations: 868)
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.541Z pid=497708 tid=b4ts class=DailyChoresCheckWorker jid=4e5ca77dfcced746f0de6ec0 INFO:   Rendered layout layouts/mailer.text.erb (Duration: 0.8ms | Allocations: 461)
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.554Z pid=497708 tid=b4ts class=DailyChoresCheckWorker jid=4e5ca77dfcced746f0de6ec0 elapsed=0.058 INFO: fail
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.554Z pid=497708 tid=b4ts WARN: {"context":"Job raised exception","job":{"retry":1,"queue":"default","class":"DailyChoresCheckWorker","args":[],"jid":"4e5ca77dfcced746f0de6ec0","created_at":1731272675.4522564,"enqueued_at":1731272683.4966125}}
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.555Z pid=497708 tid=b4ts WARN: Errno::ECONNREFUSED: Connection refused - connect(2) for nil port 587
Nov 10 21:04:43 localhost sidekiq[497708]: 2024-11-10T21:04:43.555Z pid=497708 tid=b4ts WARN: /home/ubuntu/cashflow/deployments/api-gems/bundle/ruby/3.2.0/gems/net-smtp-0.5.0/lib/net/smtp.rb:663:in `initialize'
```

Este:
```
Errno::ECONNREFUSED: Connection refused - connect(2) for nil port 587
```

Pasaba que no tenía las envs en el archivo que configuré para eso:
```bash
# scripts/sidekiq.service

EnvironmentFile=/home/ubuntu/.sidekiq_envs
```

Una vez las agregué:
```
MAILGUN_DOMAIN=""
```

reinicie el servidor:
```
systemctl --user restart sidekiq.service
```

Y ahí sí salieron los correos:
```bash
Nov 10 21:23:16 localhost sidekiq[499442]: 2024-11-10T21:23:16.515Z pid=499442 tid=b4ku class=DailyChoresCheckWorker jid=1b7dba7ffe92b4b9656c2b90 INFO: start

Nov 10 21:23:16 localhost sidekiq[499442]: 2024-11-10T21:23:16.544Z pid=499442 tid=b4ku class=DailyChoresCheckWorker jid=1b7dba7ffe92b4b9656c2b90 INFO:   Rendered layout layouts/mailer.html.erb (Duration: 1.1ms | Allocations: 855)

Nov 10 21:23:16 localhost sidekiq[499442]: 2024-11-10T21:23:16.545Z pid=499442 tid=b4ku class=DailyChoresCheckWorker jid=1b7dba7ffe92b4b9656c2b90 INFO:   Rendered layout layouts/mailer.text.erb (Duration: 0.5ms | Allocations: 461)

Nov 10 21:23:18 localhost sidekiq[499442]: 2024-11-10T21:23:18.709Z pid=499442 tid=b4ku class=DailyChoresCheckWorker jid=1b7dba7ffe92b4b9656c2b90 INFO:   Rendered layout layouts/mailer.html.erb (Duration: 0.4ms | Allocations: 237)

Nov 10 21:23:18 localhost sidekiq[499442]: 2024-11-10T21:23:18.710Z pid=499442 tid=b4ku class=DailyChoresCheckWorker jid=1b7dba7ffe92b4b9656c2b90 INFO:   Rendered layout layouts/mailer.text.erb (Duration: 0.2ms | Allocations: 212)

Nov 10 21:23:19 localhost sidekiq[499442]: 2024-11-10T21:23:19.550Z pid=499442 tid=b4ku class=DailyChoresCheckWorker jid=1b7dba7ffe92b4b9656c2b90 elapsed=3.035 INFO: done
```