# Error de Nokogiri al crear imagen Docker

Me topé con este error al intenter crear una imagen que usaba Ruby 3.0.2 Alpine 3.14

```
Caused by:
LoadError: Error loading shared library ld-linux-aarch64.so.1: No such file or directory (needed by /usr/local/bundle/gems/nokogiri-1.15.4-aarch64-linux/lib/nokogiri/3.0/nokogiri.so) - /usr/local/bundle/gems/nokogiri-1.15.4-aarch64-linux/lib/nokogiri/3.0/nokogiri.so
```

No sé porque explotaba eso específicamente en mi Docker local y no en el CI.

La solución la menciona la [documentación de Nokogiri](https://nokogiri.org/tutorials/installing_nokogiri.html#linux-musl-error-loading-shared-library). Hay que instalar gcompat.

Comando de Alpine:
```bash
apk add gcompat
```

Instrucción en Docker:
```bash
RUN set -ex \
    && apk update \
    && apk add build-base \
         postgresql-dev \
         tzdata \
         nodejs \
         git \
         libxml2-dev \
         libxslt-dev \
         gcompat \
         openssl \
         yarn
```

# Error objc69365: +NSNumber initialize al consultar en rails console

Cuando hacía alguna consulta en la consola de Rails me daba este error:

```
objc[69365]: +[NSNumber initialize] may have been in progress in another thread when fork() was called.
objc[69365]: +[NSNumber initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead. Set a breakpoint on objc_initializeAfterForkError to debug.
```

y me sacaba de la consola.

La solución que encontré de momento es desactivar spring mediante variable:

```
export DISABLE_SPRING=true

rcon

[1] pry(main)> Account.first
  Account Load (814.9ms)  SELECT "accounts".* FROM "accounts" ORDER BY "accounts"."id" ASC LIMIT $1  [["LIMIT", 1]]
```

Visto en [Stack Overflow](https://stackoverflow.com/a/68910843/1407371).

# Pattern Matching en Ruby

Tomado del artículo de [Andres Chacon](https://a-chacon.com/en/ruby/tip/2023/12/08/ruby-tip-pattern-matching.html).

> Pattern Matching in Ruby allows for concise data destructuring, making it easy to assign variables with clear syntax. From filtering values in arrays to customizing destructuring in classes, this feature simplifies data manipulation in an elegant way.

Se puede usar desde Ruby 3.0. [Documentación](https://docs.ruby-lang.org/en/master/syntax/pattern_matching_rdoc.html).

Sintaxis:
```ruby
case <expression>
in <pattern1>
  # ...
in <pattern2>
  # ...
else
  # ...
end
```

Un patrón puede ser un:
- Valor: cualquier objeto ruby. Se compara usando ===
- patrón Array: `[patron, patron, patron]`
- patrón Find:  `[*variable, subpatron]`
- patrón Hash: `{key: subpatron}`
- Combinación de patrones con barra vertical `|`
- Captura de variable: `<pattern> => variable`

## Ejemplo en Replit

Replit -> https://replit.com/@cesc89/PatternMatching?v=1

```ruby
case data
in [1]
  puts "Soy un array"
in [1,2]
  puts "array con dos elementos"
in {a: Integer}
  puts "soy un hash"
in :ok
  puts "soy un simbolo"
in [Integer, *]
  puts "soy un array de enteros"
else
  puts "no machea"
end
```

Según diferentes ejemplo veamos:

```ruby
data = [1,2]
# => array con dos elementos

data = [1,2,3,4,5,6,7,8,9,10]
# => soy un array de enteros

data = {a: 2, b: 1}
# => soy un hash

data = :ok
# => soy un simbolo
```

Así se usa la búsqueda por patrones:
```ruby
ary = [1,2,:ok, :err]

case ary
in [*, Symbol, *]
  puts "macheo"
else
  puts "sin macheo"
end

# => macheo
```

## Enlazando Variables

Esta es la forma de asignar un valor que hace matcha a una variable.

```ruby
case [1, 2]
in Integer => uno, Integer => dos
  puts "matched: #{uno} y #{dos}"
else
  puts "not matched"
end

# => matched: 1 y 2
```

## Conclusión

Está bien bacano esto en Ruby. Pattern matching es una de las cosas chéveres de Elixir y me parece bacano que esté disponible en Ruby.

# Error aws-sdk-core/ini_parser.rb:28

Quería ejecutar unas rake tasks que se conectan con AWS Athena y me salía este error:
```bash
$ be rake request_execution:request_all
rake aborted!
NoMethodError: undefined method `[]' for nil:NilClass
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/ini_parser.rb:28:in `block in ini_parse'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/ini_parser.rb:13:in `each'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/ini_parser.rb:13:in `inject'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/ini_parser.rb:13:in `ini_parse'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/shared_config.rb:420:in `load_credentials_file'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/shared_config.rb:60:in `initialize'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core.rb:128:in `new'
/Users/francisco/.gem/ruby/3.0.2/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core.rb:128:in `shared_config'
```

Pensé que sería algo de las credenciales pero no fue así. Probé revisando los archivos .env pero tampoco funcionó. Lo que me dio una pista fue [este issue](https://github.com/aws/aws-sdk-ruby/issues/837) donde dice que el sdk de AWS no estaba leyendo bien el archivo `~/.aws/credentials`.

Así se veía el archivo:
```
[default]
    aws_access_key_id = KEY
    aws_secret_access_key = SECRET
```

Al cambiarlo a esta forma:
```
[default]
aws_access_key_id = KEY
aws_secret_access_key = SECRET
```

Pude ejecutar normalmente todo.

¿raro?

# Comparando Fechas en Ruby y Rails

Quería comprobar si una fecha estaba en un rango de fechas.

Ejemplo del rango:
```ruby
(initial_visit_date - 7.days)..initial_visit_date
```

Puedo saber si una fecha X está dentro de ese rango con dos métodos:
- `Range#cover?` -> https://ruby-doc.org/core-3.1.0/Range.html#method-i-cover-3F
- `Comparable#between?` -> https://ruby-doc.org/core-3.0.0/Comparable.html#method-i-between-3F

Ejemplo con `cover?`
```ruby
seven_day_limit.cover?(patient_last_login_to_app)
```

Arrojará `true` si la fecha está dentro del rango.

Ejemplo con `between?`
```ruby
patient_last_login_to_app.between?(initial_visit_date - 7.days, initial_visit_date)
```

Este método se puede usar porque la clase Date incluye al módulo Comparable.

Encontré sobre estos dos métodos en [Stack Overflow](https://stackoverflow.com/questions/4521921/how-to-know-if-todays-date-is-in-a-date-range).

# Pruebas en rspec para envío de emails

Quería probar que un correo saliera. El mailer se ejecuta así:

```ruby
PatientMailer.updated_exercise_program(patient, self).deliver_later
```

Y en las pruebas tenía algo como:

```ruby
deliveries = ActionMailer::Base.deliveries

expect(deliveries.count).not_to eq(0)
expect(deliveries.count).to eq(2)
expect(deliveries.last.to.first).to include("@random.com")
expect(deliveries.last.body.to_s).to include("evw")
```

Pero fallaban porque  con `deliver_later` el correo se envía desde segundo plano.

Entonces intenté algo como esto que vi en otro archivo:
```ruby
RSpec.describe ExerciseProgram, type: :model do
	expect(ActionMailer::MailDeliveryJob).to have_been_enqueued.with(
	  "PatientMailer",
	  "updated_exercise_program",
	  "deliver_now",
	  args: [program.patient, program]
	)
end
```

pero fallaba con este error:
```
StandardError:
  To use ActiveJob matchers set `ActiveJob::Base.queue_adapter = :test`
```

¿por qué en el test que encontré si servía?

```ruby
RSpec.describe "Send something" do
	expect(ActionMailer::MailDeliveryJob).to have_been_enqueued.with(
		"PatientMailer",
		"patient_dashboard_app_download",
		"deliver_now",
		args: [patient, "12345", "https://link.com/apps"]
	)
end
```

## Conclusión

Encontré recomendaciones similares en Stack Overflow.

Una recomienda pruebas unitarias al mailer y otra a la integración pero en este caso quería hacer integración y no funcionó sino cambiando el método `deliver_later` a `deliver_now`.

**Enlaces**

- [How to test ActionMailer deliver_later with rspec](https://stackoverflow.com/questions/27647749/how-to-test-actionmailer-deliver-later-with-rspec)
- [How to check what is queued in ActiveJob using Rspec](https://stackoverflow.com/questions/26274954/how-to-check-what-is-queued-in-activejob-using-rspec)


# Strong Parameters elimina valores nulos de parámetro que es un Array

Tenemos configurado una nueva API de PSR para que reciba un array de valores. Así luce un JSON:

```json
{
  component_id: act.id.to_s,
  field_name: 'aggravating_activities',
  value: ["Hola", 10, false]
}
```

A esta estructura le encontramos un problema cuando se manda de esta forma:

```json
{
  component_id: act.id.to_s,
  field_name: 'aggravating_activities',
  value: [nil, nil, false]
}
```

Cuando se envía así, Strong Parameters de Rails limpia el array `value` dejándolo de esta forma:

```
value: [false]
```

Según leí en [este](https://github.com/rails/rails/issues/13766) y este [otro issue](https://github.com/rails/rails/issues/13420), es un tema de seguridad de Rails desde hace muchos años. Así que esa estructura la puedo usar desactivando la configuración de [deep_munge](https://github.com/imanel/rails/commit/060c91cd59ab86583a8f2f52142960d3433f62f5) o verificar el array en pasos intermedios para dejarlo como lo espero.

## Soluciones

Le pregunté a ChatGPT y me sugirió una solución agregando un middleware para evitar que se pierdan los nil.

```ruby
config.middleware.insert_before ActionDispatch::ParamsParser, Middleware::KeepNilValues
```

No me interesa tanto ir por ahí.

También se puede desactivar pero queda expuesto a posibles vulnerabilidades. Se desactiva así:

```ruby
config.action_dispatch.perform_deep_munge = false
```

## Otros Detalles

Esto ya lo había descubierto pero en el mundo de las pruebas en [[Apuntes_Testing_en_Ruby_y_Rails_Parte_2]]

En [este PR de 2014](https://github.com/rails/rails/pull/16924) se actualiza deep_munge para que un Array de nulos se convierta en un array vacío.

```ruby
ary = [nil, nil, nil]
# => []
```

En la sección de Seguridad de las [guías de Rails](https://edgeguides.rubyonrails.org/security.html#unsafe-query-generation) se detalla las formas en que los parámetros son limpiados por deep_munge si llevan valores nulos.

Esta es la tabla presente en dicha sección de las guías.

| JSON                              | Parameters               |
|-----------------------------------|--------------------------|
| `{ "person": null }`              | `{ :person => nil }`     |
| `{ "person": [] }`                | `{ :person => [] }`     |
| `{ "person": [null] }`            | `{ :person => [] }`     |
| `{ "person": [null, null, ...] }` | `{ :person => [] }`     |
| `{ "person": ["foo", null] }`     | `{ :person => ["foo"] }` |

# Error de gema PG al desinstalar PostgreSQL - Homebrew

Quería instalar PostgreSQL version 16 y ya tenía la 14 instalada. Así que seguí los pasos normales de instalar y desinstalar. Cuando intenté volver a configurar el proyecto Patient Self Report me encontré con este error:

```bash
$ rails db:setup
Calling `DidYouMean::SPELL_CHECKERS.merge!(error_name => spell_checker)' has been deprecated. Please call `DidYouMean.correct_error(error_name, spell_checker)' instead.
rails aborted!
LoadError: dlopen(/Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg_ext.bundle, 0x0009): Library not loaded: /opt/homebrew/opt/postgresql@14/lib/postgresql@14/libpq.5.dylib
  Referenced from: <F948A258-B067-3468-89EA-C8EC5FD77B77> /Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg_ext.bundle
  Reason: tried: '/opt/homebrew/opt/postgresql@14/lib/postgresql@14/libpq.5.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OS/opt/homebrew/opt/postgresql@14/lib/postgresql@14/libpq.5.dylib' (no such file), '/opt/homebrew/opt/postgresql@14/lib/postgresql@14/libpq.5.dylib' (no such file), '/usr/local/lib/libpq.5.dylib' (no such file), '/usr/lib/libpq.5.dylib' (no such file, not in dyld cache) - /Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg_ext.bundle
/Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg.rb:49:in `require'
/Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg.rb:49:in `block in <module:PG>'
/Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg.rb:37:in `block in <module:PG>'
/Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg.rb:42:in `<module:PG>'
/Users/francisco/.gem/ruby/3.1.0/gems/pg-1.5.3/lib/pg.rb:6:in `<top (required)>'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler/runtime.rb:60:in `require'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler/runtime.rb:60:in `block (2 levels) in require'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler/runtime.rb:55:in `each'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler/runtime.rb:55:in `block in require'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler/runtime.rb:44:in `each'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler/runtime.rb:44:in `require'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.2.33/lib/bundler.rb:175:in `require'
/Users/francisco/projects/luna-project/patient-forms-backend/config/application.rb:17:in `<top (required)>'
/Users/francisco/projects/luna-project/patient-forms-backend/Rakefile:4:in `require_relative'
/Users/francisco/projects/luna-project/patient-forms-backend/Rakefile:4:in `<top (required)>'
/Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/commands/rake/rake_command.rb:20:in `block in perform'
/Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/commands/rake/rake_command.rb:18:in `perform'
/Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/command.rb:51:in `invoke'
/Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/commands.rb:18:in `<top (required)>'
bin/rails:4:in `require'
bin/rails:4:in `<main>'
(See full trace by running task with --trace)
```

Al instalar la gema PG con bundle install como que quedó ligada a la versión de Postgres 14. Para corregir tuve que desinstalarla del sistema:
```bash
$ gem list pg

*** LOCAL GEMS ***

pg (1.5.4, 1.5.3, 1.3.0, 1.2.3, 0.21.0)

$ gem uninstall pg

Select gem to uninstall:
 1. pg-0.21.0
 2. pg-1.2.3
 3. pg-1.3.0
 4. pg-1.5.3
 5. pg-1.5.4
 6. All versions
> 6
Successfully uninstalled pg-0.21.0
Successfully uninstalled pg-1.2.3
Successfully uninstalled pg-1.3.0
Successfully uninstalled pg-1.5.3
```

Y volver a hacer bundle install:
```
$ bi

Fetching pg 1.5.3
Installing pg 1.5.3 with native extensions
Bundle complete! 49 Gemfile dependencies, 155 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

# Creando app rails de reproducción

Se puede crear aplicaciones rails de un solo archivo para reproducir cosas que pasan en el framework sin crear una aplicación nueva. Por ejemplo, para probar cosas de gemas yo me hice una app de prueba a partir de Puntapie. Eso es muy lento para casos más puntuales.

En el repo de Rails [hay plantillas](https://github.com/rails/rails/tree/main/guides/bug_report_templates) de esto para las diferentes subgems de Rails.

Aquí tuve una dificultad con la instalación de la gema sqlite3.

## Instalando sqlite3 para rails repro app

Cuando tuve la parte del gemfile así:
```ruby
gemfile(true) do
  source "https://rubygems.org"

  gem "rails", "7.0.4"
  gem "sqlite3"
end
```

Arrojaba este error cuando ejecutaba `ruby nested_has_one_rails7.rb`:
```bash
$ ruby nested_has_one_rails7.rb 
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.4.22/lib/bundler/rubygems_integration.rb:280:in `block (2 levels) in replace_gem': Error loading the 'sqlite3' Active Record adapter. Missing a gem it depends on? can't activate sqlite3 (~> 1.4), already activated sqlite3-2.1.0-arm64-darwin. Make sure all dependencies are added to Gemfile. (LoadError)
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/sqlite3_adapter.rb:13:in `<top (required)>'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/abstract/connection_handler.rb:268:in `require'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/abstract/connection_handler.rb:268:in `resolve_pool_config'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/abstract/connection_handler.rb:129:in `establish_connection'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_handling.rb:52:in `establish_connection'
	from nested_has_one_rails7.rb:15:in `<main>'
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.4.22/lib/bundler/rubygems_integration.rb:280:in `block (2 levels) in replace_gem': can't activate sqlite3 (~> 1.4), already activated sqlite3-2.1.0-arm64-darwin. Make sure all dependencies are added to Gemfile. (Gem::LoadError)
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/sqlite3_adapter.rb:13:in `<top (required)>'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/abstract/connection_handler.rb:268:in `require'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/abstract/connection_handler.rb:268:in `resolve_pool_config'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_adapters/abstract/connection_handler.rb:129:in `establish_connection'
	from /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_handling.rb:52:in `establish_connection'
	from nested_has_one_rails7.rb:15:in `<main>'
```

Encontré en un [issue](https://github.com/rails/rails/issues/35153#issuecomment-460455573) que la solución podría estar en algo como:
```ruby
gemfile(true) do
  source "https://rubygems.org"

  gem "rails", "7.0.4"
  gem "sqlite3", "`~> 1.3.6`"
end
```

Sin embargo, eso también me daba error:
```bash
$ ruby nested_has_one_rails7.rb 
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
Fetching sqlite3 1.3.13
Installing sqlite3 1.3.13 with native extensions
/Users/francisco/.gem/ruby/3.1.0/gems/bundler-2.4.22/lib/bundler/installer/parallel_installer.rb:164:in `handle_error': Gem::Ext::BuildError: ERROR: Failed to build gem native extension. (Bundler::InstallError)
\e[0m
    current directory: /Users/francisco/.gem/ruby/3.1.0/gems/sqlite3-1.3.13/ext/sqlite3
/Users/francisco/.rubies/ruby-3.1.0/bin/ruby -I /Users/francisco/.rubies/ruby-3.1.0/lib/ruby/3.1.0 -r ./siteconf20240924-38634-bp0p3o.rb extconf.rb
checking for sqlite3.h... yes
checking for pthread_create() in -lpthread... yes
checking for sqlite3_libversion_number() in -lsqlite3... yes
checking for rb_proc_arity()... yes
checking for rb_integer_pack()... yes
checking for sqlite3_initialize()... yes
checking for sqlite3_backup_init()... yes
checking for sqlite3_column_database_name()... yes
checking for sqlite3_enable_load_extension()... no
checking for sqlite3_load_extension()... no
checking for sqlite3_open_v2()... yes
checking for sqlite3_prepare_v2()... yes
checking for sqlite3_int64 in sqlite3.h... yes
checking for sqlite3_uint64 in sqlite3.h... yes
creating Makefile
```

Intenté instalar así pero tampoco funcionó:
```bash
$ gem install sqlite3 -v 1.3.13 -- --disable-march-tune-native
Building native extensions with: '--disable-march-tune-native'
This could take a while...
ERROR:  Error installing sqlite3:
	ERROR: Failed to build gem native extension.

    current directory: /Users/francisco/.gem/ruby/3.1.0/gems/sqlite3-1.3.13/ext/sqlite3
/Users/francisco/.rubies/ruby-3.1.0/bin/ruby -I /Users/francisco/.rubies/ruby-3.1.0/lib/ruby/3.1.0 -r ./siteconf20240924-39089-i27n0h.rb extconf.rb --disable-march-tune-native
checking for sqlite3.h... yes
checking for pthread_create() in -lpthread... yes
checking for sqlite3_libversion_number() in -lsqlite3... yes
checking for rb_proc_arity()... yes
checking for rb_integer_pack()... yes
checking for sqlite3_initialize()... yes
checking for sqlite3_backup_init()... yes
checking for sqlite3_column_database_name()... yes
checking for sqlite3_enable_load_extension()... no
checking for sqlite3_load_extension()... no
checking for sqlite3_open_v2()... yes
checking for sqlite3_prepare_v2()... yes
checking for sqlite3_int64 in sqlite3.h... yes
checking for sqlite3_uint64 in sqlite3.h... yes
creating Makefile
```

Finalmente, opté por usar una versión ya instalada en el sistema:
```bash
$ gem list sqlite3

*** LOCAL GEMS ***

sqlite3 (2.1.0 arm64-darwin, 1.7.0 arm64-darwin, 1.6.7 arm64-darwin, 1.6.3 arm64-darwin)
```

```ruby
gemfile(true) do
  source "https://rubygems.org"

  gem "rails", "7.0.4"
  gem "sqlite3", "1.7.0"
end
```

No logré dar con la solución.

# Mejor usar sintaxis de bloque con Rails Logger

De este artículo: https://willj.net/posts/you-should-use-the-rails-logger-block-syntax/

Dice que es mejor usar la sintaxis de bloque cuando se quiera escribir al Logger de Rails para mejor el consumo de memoria de Ruby.

## Resumen

> Passing strings to the Rails logger methods (eg. Rails.logger.info(…)) **causes unnecessary object allocations**, and if you’re calling methods to generate data for your log messages then it can cause unnecessary CPU work too.

## Why this happens

> When we call `Rails.logger.debug` with a string, say `"Hello #{name}, here's some #{generated_data}!"`, the string is created as an object, including any interpolation that is required which may include methods that are called to retrieve or generate the data which also allocates objects.

```ruby
# Set the log level
> Rails.logger.level = :info
=> :info

# Do the logging
> stats = AllocationStats.trace {
	Rails.logger.debug { "Processed API request to \
      host #{somehost} for user #{user.name} (#{user.id}): status: #{request_status} \
      user_details: #{user.to_json}" }
}
=> [#<AllocationStats::Allocation:0x000000015489ffa8 @object=#<Proc:0x000000015471ea58 /Users/will/.rbenv/versions/3.3.0/lib/ruby/gems/3.3.0/gems/activesupport-7.1.3.4/lib/active_support/broadcas...

# Get an allocation count, source paths shortened for brevity
> puts stats.allocations.group_by(:sourcefile, :sourceline, :class).to_text
                  sourcefile                          sourceline  class  count
----------------------------------------------------  ----------  -----
acti7.1.3.4/lib/active_support/broadcast_logger.rb        231     Proc    2
(irb)                                                      51     Array   1
```

> This works because when you pass a block to the logger method (debug in this case) the block will only be called if the log level is currently being logged.

# Correr dos servidores Rails de una misma aplicación

> [!Note]
> Quise hacer esto para poder probar las peticiones que se enviaban al servicio Patient Forms que vivirá dentro de Edge en Luna.
> 
> Si no levantaba un 2do servidor, el inicial moría con timeout.

Necesito estar en la misma carpeta dos veces para lanzar dos veces el comando `rails server` especificando dos puertos diferentes.

**Terminal 1**
```
cd ~/projects/backend

bundle exec rails server
```

Este va a correr en el puerto 3000.

**Terminal 2**
```
cd ~/projects/backend

bundle exec rails server -p 3006 -P tmp/pids/segundo.pid
```

Este va a correr en el puerto 3006 y adicional específico que el archivo del PID (process ID) estará en la carpeta `tmp/pids`.

Tengo que especificar el archivo o sino Rails tratará de usar el por defecto `tmp/pids/server.pid`.

# Entendiendo `inverse_of` en asociaciones

Resulta que en el upgrade a Rails 7.1.4 de Edge encontré con un error causado por tener estas asociaciones así:
```ruby
# Therapist model

has_many :conversable_patients, lambda { |therapist|
    therapist
    .patients
    .without_active_waitlist_entry
    .where(id: therapist
      .appointments
      .joins(:episode)
      .active
      .where(scheduled_date: 60.days.ago..)
      .select(:patient_id))
    .unscope(where: :therapist_id)
    .distinct
  }, class_name: "Patient", inverse_of: "conversable_team"

# Patient model

has_many :conversable_team, lambda { |patient|
    unscope(where: :patient_id)
    .where(id: patient.therapists.filter { |therapist| therapist.conversable_patients.include?(patient) })
  }, class_name: "Therapist"
```

El error que causaban era que rails trataba de buscar una columna `patient_id` en la tabla patients en lugar de buscar `id`.

Ver en [[Upgrade to 7.1.4 - Notes#PG UndefinedColumn ERROR column patients.patient_id does not exist for conversable_patients]]

La solución está en quitar el `inverse_of`. ¿Por qué? ¿Qué hace esa opción?

Una explicación clara está en este [artículo en Viget](https://www.viget.com/articles/exploring-the-inverse-of-option-on-rails-model-associations/)