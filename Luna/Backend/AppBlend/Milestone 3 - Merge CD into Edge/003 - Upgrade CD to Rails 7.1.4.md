# Upgrade Clinical Dashboard to Rails 7.1.4

# Error con gema debug

Daba este error:
```bash
Failure/Error:
       def reset(prompt = '', encoding:)
         super
         Signal.trap(:SIGWINCH, nil)

     ArgumentError:
       missing keyword: :encoding
     # /Users/francisco/.gem/ruby/3.1.6/gems/debug-1.8.0/lib/debug/console.rb:25:in `reset'
```

Al usar `debugger` en algún archivo de pruebas. Tocó hacer upgrade a la [versión 1.10.0](https://github.com/ruby/debug/releases/tag/v1.10.0).

# Invalid HTTP status: :unprocessable_entity

¿Qué? Llevo un montón de años usando ese estado con [ese símbolo](https://gist.github.com/mlanett/a31c340b132ddefa9cca) y ahora resulta que no es válido? Pues sí.

En Rack 3 el nombre cambió a "Unprocessable Content". Puedo ver que la respuesta de la petición en la prueba sigue devolviendo 422 pero ahora Rack usa otro nombre:
```ruby
Rack::Utils::HTTP_STATUS_CODES
{
 # ...
 421=>"Misdirected Request",
 422=>"Unprocessable Content",
 423=>"Locked",
 424=>"Failed Dependency",
 # ...
}
```

Una vez lo cambio a `unprocessable_content` las pruebas pasan.

Explicado en [rspec-rails](https://github.com/rspec/rspec-rails/issues/2763).

# Error al cargar `app/lib/periodic_jobs.rb` en inicializador

Clinical Dashboard *no puede cargar tal archivo en el inicializador de Sidekiq*. La configuración es la misma que en Edge pero ahí no da problema.

Este era el error:
```
bundler: failed to load command: sidekiq (/Users/francisco/.gem/ruby/3.1.6/bin/sidekiq)

/Users/francisco/.gem/ruby/3.1.6/gems/bootsnap-1.16.0/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:17:in `require': cannot load such file -- periodic_jobs (LoadError)
```

¿Por qué no da problema en Edge? Por que en CD se está cargando los defaults para Rails 7.1. En cambio en Edge para Rails 6.0:
```ruby
module ClinicalDashboardBackend
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.1
  end
end

module LunaApi
  class Application < Rails::Application
    # Initialize configuration defaults for Rails 6.0 and below.
    config.load_defaults 6.0
  end
end
```

## eagerload_paths y autoload_paths

Probé esto en `config/application.rb` pero no funcaba:
```ruby
config.autoload_paths << Rails.root.join("app/lib")
config.eager_load_paths << Rails.root.join("app/lib")
```

Esto salía con o sin esas líneas:
```ruby
ap ActiveSupport::Dependencies.autoload_paths
[
    [ 0] "/Users/francisco/projects/luna-project/provider-portal/lib",
    [ 1] "/Users/francisco/projects/luna-project/provider-portal/app/controllers",
    [ 2] "/Users/francisco/projects/luna-project/provider-portal/app/controllers/concerns",
    [ 3] "/Users/francisco/projects/luna-project/provider-portal/app/exceptions",
    [ 4] "/Users/francisco/projects/luna-project/provider-portal/app/helpers",
    [ 5] "/Users/francisco/projects/luna-project/provider-portal/app/lib",
    [ 6] "/Users/francisco/projects/luna-project/provider-portal/app/mailers",
    [ 7] "/Users/francisco/projects/luna-project/provider-portal/app/models",
    [ 8] "/Users/francisco/projects/luna-project/provider-portal/app/models/concerns",
    [ 9] "/Users/francisco/projects/luna-project/provider-portal/app/services",
    [10] "/Users/francisco/projects/luna-project/provider-portal/app/workers",
    [11] "/Users/francisco/.gem/ruby/3.1.6/gems/sentry-rails-5.10.0/app/jobs",
    [12] "/Users/francisco/.gem/ruby/3.1.6/gems/devise-4.8.1/app/controllers",
    [13] "/Users/francisco/.gem/ruby/3.1.6/gems/devise-4.8.1/app/helpers",
    [14] "/Users/francisco/.gem/ruby/3.1.6/gems/devise-4.8.1/app/mailers"
]
```

¿Por qué si el directorio está ahí no se puede cargar el archivo?

Posibles causas:

- Zeitwerk no reconoce el archivo
- Sidekiq está cargando antes que Rails

## Usando callback de inicialización

Con los callbacks de inicialización de Rails se puede hacer que se espera a la carga completa del framework.

Probé esto y tampoco funcionó:
```ruby
Rails.application.config.after_initialize do
  require "periodic_jobs"
  config.periodic(&PeriodicJobs::PERIODIC_JOBS)
end
```

En cambio esto sí funcionó:
```ruby
Rails.application.config.after_initialize do
  Rails.application.eager_load!

  config.periodic(&PeriodicJobs::PERIODIC_JOBS)
end
```

¿Por qué?

Según chatgpt, la carpeta `app/lib`, para Rails, no contiene código que normalmente sería de la aplicación por lo tanto no hace tanto esfuerzo en que se cargue como el resto de carpetas.

> [!Note]
> A falta de confirmación, me parece que gpt está confundieno `./lib` con `app/lib`



## Solución: requerir con ruta absoluta

Tocó hacer así para solventar:
```ruby
require Rails.root.join("app/lib/periodic_jobs.rb")
```

