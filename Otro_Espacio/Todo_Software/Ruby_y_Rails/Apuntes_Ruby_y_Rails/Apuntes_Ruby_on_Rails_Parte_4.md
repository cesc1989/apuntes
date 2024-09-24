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