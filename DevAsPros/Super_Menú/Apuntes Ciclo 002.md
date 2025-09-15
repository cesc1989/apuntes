# Apuntes Ciclo 002

# Devise::SessionsController hereda de ApplicationController

Tuve un problema con una prueba para el controlador de sesión de API:
```bash
 1) Sessions API POST /api/v1/users/sign_in when credentials are valid returns a JWT token and user information
     Failure/Error: Current.restaurant = current_user.restaurant

     NoMethodError:
       undefined method `restaurant' for nil:NilClass
     # ./app/controllers/application_controller.rb:14:in `set_current_restaurant'
```

pero ese método no estaba definido en el controlador base para la API sino en `ApplicationController`. El fix fue simplemente hacer un skip del callback.

Creí que el problema era el controlador base:
```ruby
module Api
  class BaseApiController < ActionController::Base
  end
end
```

Estaba mirando donde no era.

> [!Warning]
> Esto es un problema de Puntapie. El controlador `Api::V1::Users::SessionsController` no está heredando de `Api::BaseApiController`.

Estaba confundido ahí al creer que el `BaseApiController` era el que gestionaba el inicio de sesión.

En todo caso, ¿por qué dicho controlador tenía acceso al método?

Resulta que `Devise::SessionsController` hereda de `ApplicationController`:
```ruby
ap  Devise::SessionsController.ancestors
[
    [ 0] Devise::SessionsController < DeviseController,
    [ 1] DeviseController < ApplicationController,
    [ 2] Devise::Controllers::ScopedViews,
    [ 3] ApplicationController < ActionController::Base,
```

Y por eso se filtra el método hasta ese controlador.