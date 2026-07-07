# Apuntes Apex - Ciclo 007 - Solid Queue

## Configuración de Cash Flow

La configuración inicial es similar a los otros dos Solid. Configuré para development como para production lo que respecta a la base de datos ya esta configuración:
```ruby
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```

Para development configuré esta línea también:
```ruby
config.solid_queue.logger = ActiveSupport::Logger.new(STDOUT)
```

Ya que decidí correr el servidor de jobs con el comando que ya trae en vez de integrarlo a Puma. Así:
```bash
./bin/jobs
```

### Mission Control Jobs

Esto es lo que viene a ser Sidekiq Web. Lo puse igual en el routes, tal como dice la guía:
```ruby
authenticate :user, lambda { |u| u.admin? } do
	mount MissionControl::Jobs::Engine, at: "/cashflow-jobs"
end
```

Pero me tocó poner esta línea en `application.rb` para que pudiera usar el auth que ya tengo configurado:
```ruby
config.mission_control.jobs.http_basic_auth_enabled = false
```

### Recurring

Así quedó la configuración del archivo `config/recurring.yml`:
```yaml
recurring_expenditures_job:
  class: RecurringExpendituresJob
  schedule: "0 7 1 * *" # First day of the Month at 7am
daily_chores_check:
  class: DailyChoresCheckJob
  schedule: "0 6 * * *" # Daily at 6am
```

Toca sin llave del entorno porque sino no salen en development y no puedo confirmar la funcionalidad.

## Warning sobre forkeo

Cuando se corre el servidor de jobs sale este mensaje en los logs:
```
warning: Writable sqlite database connection(s) were inherited from a forked process. This is unsafe and the connections are being closed to prevent possible data corruption. Please close writable sqlite database connections before forking.
```

Es un warning que se explica porque Solid Queue corre en modo fork como explican las guías.

> [!Info]
> Ver guías: https://github.com/rails/solid_queue?tab=readme-ov-file#fork-vs-async-mode

## Error al cerrar foreman

En development, al cerrar foreman salía este error:
```bash
SQLite3::BusyException: database is locked (ActiveRecord::StatementInvalid: SQLite3::BusyException: database is locked)
```

La solución para esto y para quitar el warning es correr los jobs en modo async:
```bash
./bin/jobs --mode async
```

## Error de `preview_path=` al correr un Job

Pasó por la gema `rspec-rails`. Toca subirla a la versión 7.1.1 para quitar ese error.

## Error de Translation

Me dio este error al intentar correr de nuevo el job fallido anteriormente:
```
Translation missing: en.time.formats.next_time_at
```

La solución, según Claudio, fue agregar esta línea en `application.rb`:
```ruby
config.i18n.fallbacks = true
```