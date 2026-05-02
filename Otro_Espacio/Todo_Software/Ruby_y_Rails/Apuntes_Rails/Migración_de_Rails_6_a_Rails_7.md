# Migración de Rails 6 a Rails 7

# Pasos
1. Asegurarse de incrementar el nivel de *coverage* de la suite de pruebas
2. Estar en la versión de Ruby 2.7.1
3. Estar en la versión de Rails 6.1.4
4. Cambiar la versión de Rails en el Gemfile a la más reciente
5. Correr el comando `bundle update`
6. Correr el comando `rails app:update`
    1. Usar la opción `d` para ver las diferencias en los archivos
    2. Usar la opción `m` para hacer una mezcla sin borrar lo existente
        1. Se puede [configurar Sublime Text](https://stackoverflow.com/questions/44549733/how-to-use-visual-studio-code-as-default-editor-for-git-mergetool/44549734#44549734) como herramienta de *merge* para simplificar esta parte.
7. Revisa la [documentación aquí](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html) por cualquier pieza faltante
8. Actualiza las gemas faltantes según lo que [se indique en RailsDiff](https://railsdiff.org/6.0.3.6/7.0.1)


# Errores luego de migrar
## Error de ViewComponent::Compiler.mode

Este error

    rails aborted!
    NameError: uninitialized constant ViewComponent::Compiler
    
          ViewComponent::Compiler.mode = if Rails.env.development? || Rails.env.test?
                       ^^^^^^^^^^
    Did you mean?  Complex
    /Users/francisco/projects/leisure-shelf-playground/config/environment.rb:5:in `<top (required)>'

Se arregla en el Gemfile cambiando de esto:

    gem "view_component", require: "view_component/engine"

a esto

    gem "view_component"

Visto en este [issue](https://github.com/ViewComponent/view_component/issues/1468#issuecomment-1539731168).

## Error de gema pg en versión inferior

Este error

    rails aborted!
    LoadError: Error loading the 'postgresql' Active Record adapter. Missing a gem it depends on? can't activate pg (~> 1.1), already activated pg-0.21.0. Make sure all dependencies are added to Gemfile.
    
    Caused by:
    Gem::LoadError: can't activate pg (~> 1.1), already activated pg-0.21.0. Make sure all dependencies are added to Gemfile.

Solución. Cambiar la versión de la gema pg a mayor 1.1

    gem 'pg', '~> 1.1'


## Error con versión de redis y turbo-rails
    Gem::LoadError (Error loading the 'redis' Action Cable pubsub adapter. Missing a gem it depends on? can't activate redis (>= 3, < 5), already activated redis-5.0.6. Make sure all dependencies are added to Gemfile.):

La solución es tener una versión de la gema redis menor a la 5. En este caso la solución fue:

    gem "redis", "~> 4.0"

que instaló la versión:

    Using redis 4.8.1 (was 5.0.6)

