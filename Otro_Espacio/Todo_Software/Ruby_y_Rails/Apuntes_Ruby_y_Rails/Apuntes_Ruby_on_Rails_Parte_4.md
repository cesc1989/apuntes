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