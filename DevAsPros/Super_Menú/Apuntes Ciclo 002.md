# Apuntes Ciclo 002

## Devise::SessionsController hereda de ApplicationController

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

## Dependencia circular al pasar JSON a Inertia component

Usar `to_json` aquí causa una dependencia circular por las relaciones de Category con Dish.
```ruby
class PublicMenusController < BasePublicMenuController
  def show
    restaurant = Restaurant.find_by!(slug: params[:slug])
    dishes = restaurant.dishes

    render(
      inertia: "Menu",
      props: {
        dishes: dishes.to_json,
        name: restaurant.name
      }
    )
  end
end
```

Así sale en local en la consola de Rails:
```ruby
Restaurant.first.dishes.to_json
  Restaurant Load (0.2ms)  SELECT "restaurants".* FROM "restaurants" ORDER BY "restaurants"."id" ASC LIMIT ?  [["LIMIT", 1]]
  Dish Load (0.2ms)  SELECT "dishes".* FROM "dishes" INNER JOIN "categories" ON "dishes"."category_id" = "categories"."id" WHERE "categories"."restaurant_id" = ?  [["restaurant_id", 1]]
  ActiveStorage::Attachment Load (0.1ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? LIMIT ?  [["record_id", 1], ["record_type", "Dish"], ["name", "photo"], ["LIMIT", 1]]
  ActiveStorage::Blob Load (0.1ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
(irb):2:in `<main>': stack level too deep (SystemStackError)
```

Hay que pasar un serializer o la lista de atributos:
```ruby
Restaurant.first.dishes.to_json(only: [:id, :name, :description, :price, :enabled])
  Restaurant Load (0.1ms)  SELECT "restaurants".* FROM "restaurants" ORDER BY "restaurants"."id" ASC LIMIT ?  [["LIMIT", 1]]
  Dish Load (0.1ms)  SELECT "dishes".* FROM "dishes" INNER JOIN "categories" ON "dishes"."category_id" = "categories"."id" WHERE "categories"."restaurant_id" = ?  [["restaurant_id", 1]]
=> "[{\"id\":1,\"name\":\"Bruschetta\",\"description\":\"Toasted bread with tomatoes and basil\",\"price\":8500,\"enabled\":true},{\"id\":2,\"name\":\"Calamari Rings Powa\",\"description\":\"Fried squid rings with marinara sauce\",\"price\":1200,\"enabled\":true},{\"id\":3,\"name\":\"Grilled Salmon\",\"description\":\"Fresh salmon with lemon herbs\",\"price\":2500,\"enabled\":true},{\"id\":4,\"name\":\"Chicken Parmesan\",\"description\":\"Breaded chicken with marinara and cheese\",\"price\":2200,\"enabled\":true},{\"id\":5,\"name\":\"Vegetable Pasta Piwa\",\"description\":\"Penne pasta with seasonal vegetables\",\"price\":1800,\"enabled\":true},{\"id\":6,\"name\":\"Tiramisu\",\"description\":\"Classic Italian dessert\",\"price\":900,\"enabled\":true},{\"id\":7,\"name\":\"Chocolate Cake\",\"description\":\"Rich chocolate cake with vanilla ice cream\",\"price\":800,\"enabled\":true},{\"id\":8,\"name\":\"Sancocho de Mondongo\",\"description\":\"sancochazo\",\"price\":20000,\"enabled\":true},{\"id\":9,\"name\":\"Pataconales\",\"description\":\"Patacon sabrosono\",\"price\":8900,\"enabled\":true}]"
```

## Error de build de Vite

Este error en el paso de assets precompile:
```bash
Building with Vite ⚡️
vite v5.4.19 building for production...
transforming...
✓ 723 modules transformed.
rendering chunks...
[plugin @tailwindcss/vite:generate:build] Sourcemap is likely to be incorrect: a plugin (@tailwindcss/vite:generate:build) was used to transform files, but didn't generate a sourcemap for the transformation. Consult the plugin documentation for help
Killed

pid 779285 exit 137
Build with Vite failed! ❌
```

Lo mejor que encontré para solucionar de momento es desactivar el sourcemap:
```js
# vite.config.ts

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
  build: {
    sourcemap: true // o false si no necesitas sourcemaps en producción
  }
})
```

## Mejorar tiempos de npm ci

Esta vaina es lenta y está halando recursos del servidor lo que termina causando errores del despliegue.

Ejemplo:
```bash
client_loop: send disconnect: Broken pipe
Error: Process completed with exit code 255.
```

Claude recomienda usar las banderas:
```
--prefer-offline --no-audit --no-fund
```

Los dos últimas ayudan pero no mucho. La que es más clave es la primera pero tiene sus gallitos.

Lecturas:

- [Stack Overflow](https://stackoverflow.com/questions/55230628/is-there-a-way-to-speedup-npm-ci-using-cache)
- [Faste npm installs](https://www.tiernok.com/posts/2019/faster-npm-installs-during-ci)

### Quitar salida a stdout

Según [este artículo](https://jeromewu.github.io/how-to-speed-up-node-js-modules-installation/) escribir a stdout es lento. Mejor que este build no lo haga.

Probé este cambio:
```bash
if ! timeout 60 npm ci --prefer-offline --no-audit --no-fund 2>> /home/ubuntu/supermenu/deployment_logs/012_npm_install.log; then
  echo "$(date '+%F %T') npm install failed or timed out" >> /home/ubuntu/supermenu/deployment_logs/012_npm_install.log
  exit 1
else
  echo "$(date '+%F %T') npm install completed successfully" >> /home/ubuntu/supermenu/deployment_logs/012_npm_install.log
fi
```

Aún me dio error. El build tardó los 5 minutos.

```bash
added 173 packages in 6s
client_loop: send disconnect: Broken pipe
Error: Process completed with exit code 255.
```

`npm ci` está terminando en 6 segundos. El problema parece estar en el `bundle install`.

## Revisión de bundle install