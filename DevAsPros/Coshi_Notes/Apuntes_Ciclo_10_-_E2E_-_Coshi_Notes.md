# Apuntes Ciclo 10 - E2E - Coshi Notes

# Errores de gemas al subir a Rails 7.1

Me topé con estos dos errores al subir la versión de Rails.

Primero, gema base64:

    You have already activated base64 0.1.1, but your Gemfile requires base64 0.2.0. Since base64 is a default gem, you can either remove your dependency on it or try updating to a newer version of bundler that supports base64 as a default gem. (Gem::LoadError)
      /opt/rubies/ruby-3.1.0/lib/ruby/3.1.0/bundler/runtime.rb:309:in `check_for_activated_spec!'

Segundo, gema stringio:

    You have already activated stringio 3.0.1, but your Gemfile requires stringio 3.1.0. Since stringio is a default gem, you can either remove your dependency on it or try updating to a newer version of bundler that supports stringio as a default gem. (Gem::LoadError)

La solución fue fijar ambas gemas, en el Gemfile, a la versión que decía que estaba activada:

    gem 'base64', '0.1.1'
    gem 'stringio', '3.0.1'
# Passenger que muestre errores en Prod

Cuando estaba fallando, solo salía una página de error que no decía mucho. Para facilitar, uno puede cambiar el modo de passenger en el archivo de Nginx para que salgan los errores. Pero eso debe deshacerse.

Para esa labor, configuré el archivo nginx así:

      passenger_enabled on;
      passenger_ruby /opt/rubies/ruby-3.1.0/bin/ruby;
      passenger_app_env production;
    
      # Dejo este comentado para cuando necesite resolver errores en prod, descomentar y comentar la
      # instrucción de arriba.
      # passenger_app_env development;

De esta forma si hay algún fallo, entro al servidor y descomento las líneas que me interesan.

# Instalando Passenger Standalone para Development

La cosa iba bien en el proceso hasta que intenté correr el servidor de Rails

    $ bundle exec rails server
    Rack::Handler is deprecated and replaced by Rackup::Handler
    /Users/francisco/.gem/ruby/3.1.0/gems/passenger-6.0.19/src/ruby_supportlib/phusion_passenger/rack_handler.rb:96: warning: Calling Rackup::Handler.register with a string is deprecated, use the class/module itself.
    /Users/francisco/.gem/ruby/3.1.0/gems/rackup-2.1.0/lib/rackup/handler.rb:22:in `const_get': uninitialized constant Rackup::Handler::Rack (NameError)
    
            klass = self.const_get(klass, false)
                        ^^^^^^^^^^
    Did you mean?  Racc
        from /Users/francisco/.gem/ruby/3.1.0/gems/rackup-2.1.0/lib/rackup/handler.rb:22:in `register'

Salió ese error y ni volviendo a versiones anteriores se corrigió.

# Desactiva los logs en los tests

Según este trino, y también comentarios de Jason Swett y otros, es mejor desactivar logs en el entorno de pruebas. Así será más rápido todo.

Así quedó el archivo `config/environments/test.rb`:

    Rails.application.configure do
      # (otras configuraciones)
      
      if ENV['RAILS_LOG'].blank?
        config.logger = Logger.new(nil)
        config.log_level = :fatal
      end
    end


# Optimizando SQLite3

En [este artículo](https://fractaledmind.github.io/2023/09/07/enhancing-rails-sqlite-fine-tuning/) explican al detalle. De momento, seguí la configuración sugerida por este [gist](https://gist.github.com/fractaledmind/3565e12db7e59ab46f839025d26b5715/645f2d2dde3a275c270eabc00ce3067583b1b530) y estoy probando y revisando cómo va todo.


# Problemas de Sesión en Dev y en Prod

En Dev, no sé bien qué pasó.

En prod, tuvo que ver configurar `force_ssl = true`.

Más detalles en [+Devise no cierra sesión [Coshi Notes]](https://paper.dropbox.com/doc/Devise-no-cierra-sesion-Coshi-Notes-g01bLci5HvL9WzXlBAKgy) 


# Capybara y Rails System Tests

Los Rails System Tests son la nueva forma de hacer E2E tests en Rails.

RSpec los recomienda por encima de los Feature specs. Según otros artículos también son mejores porque ya el framework se encarga de varias configuraciones:

- Incluir capybara
- Incluir los webdrivers
- Limpiar la base de datos
- Tomar capturas de pantalla


## Artículos
- [A Quick Guide to Rails System Tests in RSpec](https://medium.com/table-xi/a-quick-guide-to-rails-system-tests-in-rspec-b6e9e8a8b5f6)
    - detalles generales y algunas configuraciones
- [System of a test: Proper browser testing in Ruby on Rails](https://evilmartians.com/chronicles/system-of-a-test-setting-up-end-to-end-rails-testing)
    - Muestran una forma más optima? Tocaría probar.
- [RSpec/Capybara integration tests: the ultimate guide](https://www.codewithjason.com/rails-integration-tests-rspec-capybara/)
    - En este Jason Swett se va a fondo y explica todo.


## Configuraciones

De este trabajo destaco algunas configuraciones que hice. Muchas de estas deben ir para Puntapie.

**Archivo .rspec**
Lo corregí para que quedara así:

    --require rails_helper

**Actualización de rspec-rails**
Lo subí a la versión 6.1. Dejarlo en versiones anteriores mostraba varios DEPRECATION_WARNINGS.

**Configuración de Helpers de pruebas de Devise**
Para tener ayuda con las pruebas de cosas de Devise

    # spec/support/devise_testing_helpers.rb
    
    RSpec.configure do |config|
      config.include Devise::Test::IntegrationHelpers, type: :system
    end

**Configuración de Capybara**

    # spec/support/capybara_setup.rb
    
    RSpec.configure do |config|
      config.before(:each, type: :system) do
        driven_by :rack_test, screen_size: [1440, 800]
      end
    
      config.before(:each, type: :system, js: true) do
        driven_by :selenium_chrome_headless, screen_size: [1440, 800]
      end
    end

La opción `screen_size: [1440, 800]` es clave porque sino el navegador de las pruebas de sistema se abre como en tamaño móvil y ahí los elementos del sidebar están ocultos.

