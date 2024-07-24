# Migrando a Ruby 3 - Provider Portal

Los pasos iniciales son:

- Instalar Ruby 3. Versión 3.1.2 o 3.0.2
- Cambiar la versión de Ruby en los archivos:
    - Gemfile
    - .ruby-version
    - Archivo `.circleci/config.yml` - [docs](https://circleci.com/developer/images/image/cimg/ruby)
        - Para 3.1.2 sería `cimg/ruby:3.1.2-browsers`
        - Para 3.0.2 sería `cimg/ruby:3.0.2-browsers`
    - Dockerfile y sus variantes
        - Versiones de imágenes:
            - [Para 3.1.2](https://hub.docker.com/layers/ruby/library/ruby/3.1.2-alpine3.16/images/sha256-7dcfcfa70588c4b81a6c81f47fcf6f58d486ae1159354a16f07d0ba0b8b8cfe7?context=explore): `ruby:3.1.2-alpine3.16`
            - [Para 3.0.2](https://hub.docker.com/layers/library/ruby/3.0.2-alpine3.14/images/sha256-a4d4e87f31eb226933dd5b0eaf4df0c552e33714aa29e6dc3c26135d3f174344?context=explore): `ruby:3.0.2-alpine3.14`
- Correr el comando `bundle install` y esperar que no haya errores.

# Pasos previos

Los errores principales vienen por Webpacker. Hay que pasarse a Shackapacker para poder hacer un cambio de Ruby.

A tener en cuenta:

- [[Error_de_node-gyp_y_node-sass_en_Docker_con_Python]]
- [Migrando a Ruby 3.0.2 con WKHTMLTOPDF](https://paper.dropbox.com/doc/Migrando-a-Ruby-3.0.2-con-WKHTMLTOPDF-Wa5JEKz9L74a4sepwxGMz) 

# Errores cuando intenté con Ruby 3.1.2

## Primer Problema: gema ruby_dep

    ruby_dep-1.5.0 requires ruby version >= 2.2.5, ~> 2.2, which is incompatible with the current version, ruby 3.1.2p20

**Solución**: actualizar versión de Listen al menos a la 3.7.1.

## Segundo Problema: Simplecov

    $ rspec
    An error occurred while loading rails_helper.
    Failure/Error: SimpleCov.start('rails')
    
    RuntimeError:
      coverage measurement is already setup
    # /Users/fquintero/.rvm/gems/ruby-3.1.2/gems/simplecov-0.21.2/lib/simplecov.rb:354:in `start'
    # (...)
    # ./spec/rails_helper.rb:2:in `<top (required)>'

El problema aparece en [este issue](https://github.com/simplecov-ruby/simplecov/issues/1020) y está relacionado con algo de Ruby 3. Se menciona en [este otro issue](https://github.com/simplecov-ruby/simplecov/issues/1003):

> It looks like `ruby-head` made an [update](https://github.com/ruby/ruby/pull/4856/files#diff-4fd3567b713b460fa5616259e35ab22faaa28376ea929ce0254744d6e16ffd33R78) a few days ago that caused `Coverage.start` to raise an exception if you call it more than once. Naturally, this will also cause `SimpleCov.start` to raise the same exception.

Según este PR en [otro proyecto](https://github.com/devise-security/devise-security/pull/343/files) la solución sería quitar la línea:

    require 'simplecov'
    # SimpleCov.start('rails') <- ESTA

 del archivo `rails_helper.rb`.
 
 Funciona si la quito pero otro error aparece.

## Tercer Problema: Psych

Cuando corro el comando `rspec` sale este error:

    $ rspec
    
    An error occurred while loading rails_helper.
    Failure/Error: require File.expand_path('../config/environment', __dir__)
    
    Psych::BadAlias:
      Unknown alias: default
    # /Users/fquintero/.rvm/gems/ruby-3.1.2/gems/webpacker-4.2.2/lib/webpacker/env.rb:30:in `available_environments'


## Cuarto Problema: rails server

Cuando lo ejecuto sale esto otro:

    $ rserv 
    
    /Users/fquintero/.rvm/gems/ruby-3.1.2/gems/rubocop-rails-2.0.1/lib/rubocop/cop/rails_cops.rb:7:in `remove_const': constant RuboCop::Cop::Rails not defined (NameError)
    
        remove_const('Rails') if const_defined?('Rails')
        ^^^^^^^^^^^^
            from /Users/fquintero/.rvm/gems/ruby-3.1.2/gems/rubocop-rails-2.0.1/lib/rubocop/cop/rails_cops.rb:7:in `<module:Cop>'

Se soluciona actualizando la versión de Rubocop Rails a la 2.1.0.

[Issue donde](https://github.com/rubocop/rubocop-rails/issues/79) aparece el error. [Versión liberada](https://github.com/rubocop/rubocop-rails/pull/80).

# Errores cuando probé con Ruby 3.0.2

Vimos el cuarto problema que se experimentó con la versión 3.1.2.


## Segundo problema: An error occurred while RSpec/FactoryBot/CreateList cop was inspecting

[Solución](https://github.com/rubocop/rubocop-rspec/issues/1086#issuecomment-724762933), actualizar rubocop-rspec a la 2.1.0.


## Tercer error: node-sass: Command failed

El error completo:

    error /home/circleci/provider-portal/node_modules/node-sass: Command failed.
    Exit code: 1
    Command: node scripts/build.js
    Arguments: 
    Directory: /home/circleci/provider-portal/node_modules/node-sass

A continuación, todo lo que se intentó.

Primero que nada la correspondencia de versiones de Node con respecto a node-sass

    node-sass: 4.13.0
    node: 4.14+
    
    node in circle ci: 16.13.0
    corresponding node-sass: 6.0+
    
    node local: v14.17.3
    node-sass: 4.13.0
    
    AFTER UPDATE
    
    node local: v16.13.0
    node-sass: 7.0.1

**Comandos ejecutados**


    $ yarn install

El cual falló con este error:


    /Users/fquintero/.node-gyp/16.13.0/include/node/v8-internal.h:492:38: error: no template named 'remove_cv_t' in namespace 'std'; did you mean 'remove_cv'?
                !std::is_same<Data, std::remove_cv_t<T>>::value>::Perform(data);
                                    ~~~~~^~~~~~~~~~~
                                         remove_cv
    /Library/Developer/CommandLineTools/usr/include/c++/v1/type_traits:660:50: note: 'remove_cv' declared here
    template <class _Tp> struct _LIBCPP_TEMPLATE_VIS remove_cv
                                                     ^
    1 error generated.

Se soluciona al ejecutarlo de esta forma:

    $ CXXFLAGS="--std=c++17" yarn install
    yarn install v1.22.4
    [1/4] 🔍  Resolving packages...
    [2/4] 🚚  Fetching packages...
    warning @sandipmo/find-max@1.0.4: The engine "nodejs" appears to be invalid.
    [3/4] 🔗  Linking dependencies...
    warning " > webpack-dev-server@3.11.3" has unmet peer dependency "webpack@^4.0.0 || ^5.0.0".
    warning "webpack-dev-server > webpack-dev-middleware@3.7.3" has unmet peer dependency "webpack@^4.0.0 || ^5.0.0".
    [4/4] 🔨  Building fresh packages...
    ✨  Done in 111.41s.

Visto en [Stack Overflow](https://stackoverflow.com/a/70086482/1407371).

Había que reinstalar `node-sass` cuando lo intenté:

    $ yarn remove node-sass
    yarn remove v1.22.4
    [1/2] 🗑  Removing module node-sass...
    error This module isn't specified in a package.json file.
    info Visit https://yarnpkg.com/en/docs/cli/remove for documentation about this command.

Cuando quise instalar:

    $ yarn add node-sass@6.0.1
    error /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/@rails/webpacker/node_modules/node-sass: Command failed.
    Exit code: 1
    Command: node scripts/build.js
    Arguments: 
    Directory: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/@rails/webpacker/node_modules/node-sass1

Y lo logré como más arriba con `yarn install`

    $ CXXFLAGS="--std=c++17" yarn add node-sass
    yarn add v1.22.4
    [1/4] 🔍  Resolving packages...
    [2/4] 🚚  Fetching packages...
    warning @sandipmo/find-max@1.0.4: The engine "nodejs" appears to be invalid.
    [3/4] 🔗  Linking dependencies...
    warning " > webpack-dev-server@3.11.3" has unmet peer dependency "webpack@^4.0.0 || ^5.0.0".
    warning "webpack-dev-server > webpack-dev-middleware@3.7.3" has unmet peer dependency "webpack@^4.0.0 || ^5.0.0".
    [4/4] 🔨  Building fresh packages...
    success Saved lockfile.
    success Saved 42 new dependencies.
    info Direct dependencies
    └─ node-sass@7.0.1
    info All dependencies1CXXFLAGS="--std=c++17" yarn add node-sass

**El Webpacker actual y el objetivo**

    -    "@rails/webpacker": "4.2.2",
    +    "@rails/webpacker": "5.3.0",

**Sobre el comando** `**yarn upgrade**`

- Actualiza todo los paquetes
- Si cambio la versión en el `package.json` y corro yarn installa, se actualiza todo
- Según los docs debería ser

El comando

    $ yarn upgrade @rails/webpacker@^5.0.0

Tampoco. Actualizó incluso más paquetes e igual así falló la compilación.


    $ yarn add @rails/webpacker@5.0.0
    $ yarn add @rails/webpacker@5.3.0

Llegamos al error de babel y plugins:

    ERROR in Entry module not found: Error: Can't resolve 'babel-loader' in

Según, la solución es `yarn upgrade` pero ese comando actualiza todos los paquetes y luego vienen los errores de Typescript.


# Errores luego de migración exitosa

Este código causaba errores al intentar exportar las métricas:

    CSV(write_stream, col_sep: '|') do |csv|
    
    Aws::S3::MultipartUploadError
    multipart upload failed: wrong number of arguments (given 2, expected 0..1)

Eso solía funcionar en [Ruby 2.7.7](https://ruby-doc.org/stdlib-2.7.0/libdoc/csv/rdoc/CSV.html#class-CSV-label-Shortcuts) pero cuando subimos a Ruby 3.0.2 la película cambió en la clase CSV. En la versión 3 parece no estar disponible ese atajo.

Tocó usar [directamente el método de clase](https://ruby-doc.org/stdlib-3.0.2/libdoc/csv/rdoc/CSV.html#method-c-instance) `[instance](https://ruby-doc.org/stdlib-3.0.2/libdoc/csv/rdoc/CSV.html#method-c-instance)` el cual permite como primer parámetro un string o un objeto IO.

    CSV.instance(write_stream, col_sep: '|') do |csv|

Nota que el error que daba la gema AWS en la parte:

    @s3_object.upload_stream do |write_stream|

es porque, según los docs, si no se puede completar la transmisión de datos, este explota con la excepción capturada. El error de CSV hacía que explotara por otro lado.

    Raises:
            - (MultipartUploadError) — If an object is being uploaded in parts, and the upload can not be completed, then the upload is aborted and this error is raised. The raised error has a #errors method that returns the failures that caused the upload to be aborted.

