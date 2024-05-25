# Cómo Agregar Vistas Dummy
Las vistas para organizar el maquetado de la plantilla AdminLTE antes de ser implementadas en el backend.

## Agregar Ruta

El primer paso es agregar una ruta en el archivo `config/routes.rb` dentro del bloque que identifica el recurso `dummy`:

    # config/routes.rb
    
    Rails.application.routes.draw do
      resources :dummy, only: [] do
        collection do
          get :lista_pqr
        end
      end
    end
> Cuando se modifica este archivo, hay que reiniciar el servidor de Rails.

Dentro del bloque `collection do end` se agrega la ruta con la forma `get :nombre_ruta`. Ejemplo:

    collection do
      get :lista_pqr
      get :detalle_pqr
      get :crear_pqr
    end


## Definir la acción de la ruta en el controlador

Con la ruta definida, podemos definir la acción que le dará manejo a las peticiones para esa ruta. Lo hacemos en el archivo `app/controllers/dummy_controller.rb`.

Y se modifica agregando un método con el mismo nombre de la ruta definida en `config/routes.rb`.

    # Ruta en config/routes.rb
    
    get :detalle_pqr
    
    # Acción en controlador
    # app/controllers/dummy_controller.rb
    class DummyController < ApplicationController
      def lista_pqr; end
    end


## Crear vista para el código HTML

Finalmente, para poder ver el código HTML en acción hay que crear una vista. Para esto, creamos un archivo con extensión `.html.erb` en la carpeta `app/views/dummy/`.


## Ver el resultado

Con todo lo anterior completado, lanzamos el servidor rails del proyeto

    $ foreman start

Y vamos a `localhost:5000/dummy/nombre_ruta`.

