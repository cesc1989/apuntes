# Fallo al “recordar me” al iniciar sesión - Coshi Notes

## Actualización, 19 de Enero

Tuve que bajar a Rails 7.0.8 para que funcionara el chulito al hacer login.


----------

 
Después de la actualización a Rails 7 varias cosas se rompieron. Una de ellas es la opción de “Remember me” de Devise.

Cuando selecciono el chulo para que recuerde la sesión, la petición hace una autenticación, seguido una redirección a la página principal pero es respondida con un 401 y finalmente devuelto a la página de login.

## Configuraciones y versiones

Las versiones de las librerías:

- Rails: 7.1.0
- Devise 4.9.3

Todas las configuraciones están en orden.

    # config/devise.rb
    config.remember_for = 4.weeks
    
    # app/models/user.rb
    devise :masqueradable,
             :rememberable,
             :validatable
             
    # == Schema Information
    #  remember_created_at    :datetime




# Falla al chulear “recuerda me”
    Started POST "/users/sign_in" for 127.0.0.1 at 2024-01-19 10:52:14 -0500
    Processing by Users::SessionsController#create as TURBO_STREAM
      Parameters: {"authenticity_token"=>"[FILTERED]", "user"=>{"email"=>"frajaquico@aol.com", "password"=>"[FILTERED]", "remember_me"=>"1"}, "commit"=>"Iniciar sesión"}
      User Load (0.5ms)  SELECT "users".* FROM "users" WHERE "users"."email" = ? ORDER BY "users"."id" ASC LIMIT ?  [["email", "frajaquico@aol.com"], ["LIMIT", 1]]
      TRANSACTION (0.1ms)  begin transaction
      User Update (2.6ms)  UPDATE "users" SET "remember_created_at" = ?, "updated_at" = ? WHERE "users"."id" = ?  [["remember_created_at", "2024-01-19 15:52:14.875778"], ["updated_at", "2024-01-19 15:52:14.876509"], ["id", 1]]
      TRANSACTION (0.5ms)  commit transaction
    Redirected to http://localhost:3001/
    Completed 303 See Other in 457ms (ActiveRecord: 3.7ms | Allocations: 10522)
    
    
    Started GET "/" for 127.0.0.1 at 2024-01-19 10:52:14 -0500
    Processing by ChannelsController#index as TURBO_STREAM
    Completed 401 Unauthorized in 1ms (ActiveRecord: 0.0ms | Allocations: 735)
    
    
    Started GET "/users/sign_in" for 127.0.0.1 at 2024-01-19 10:52:14 -0500
    Processing by Users::SessionsController#new as TURBO_STREAM
      Rendering layout layouts/login.html.erb
      Rendering devise/sessions/new.html.erb within layouts/login

En el log se puede ver:

- se hace la petición para iniciar sesión
- se guarda en base de datos el valor para `remember_created_at`
- se redirecciona a la página principal
- se obtiene una respuesta 401
- se redirecciona a la página de inicio de sesión


# Inicia sesión sin chulear “recuerda me”

Y así se ve un flujo normal sin chulear el checkbox:

    Started POST "/users/sign_in" for 127.0.0.1 at 2024-01-19 11:03:27 -0500
    Processing by Users::SessionsController#create as TURBO_STREAM
      Parameters: {"authenticity_token"=>"[FILTERED]", "user"=>{"email"=>"frajaquico@aol.com", "password"=>"[FILTERED]", "remember_me"=>"0"}, "commit"=>"Iniciar sesión"}
      User Load (3.4ms)  SELECT "users".* FROM "users" WHERE "users"."email" = ? ORDER BY "users"."id" ASC LIMIT ?  [["email", "frajaquico@aol.com"], ["LIMIT", 1]]
    Redirected to http://localhost:3001/
    Completed 303 See Other in 425ms (ActiveRecord: 5.1ms | Allocations: 3825)
    
    
    Started GET "/" for 127.0.0.1 at 2024-01-19 11:03:28 -0500
    Processing by ChannelsController#index as TURBO_STREAM
      User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 1], ["LIMIT", 1]]
    Rendering layout layouts/application.html.erb
    Rendering channels/index.html.erb within layouts/application

Y se puede ver que:

- se hace la petición para iniciar sesión
- se redirecciona a la página principal
- y se cargan los canales


# Sí se crean las cookies, sigue sin iniciar sesión

La de remember_me

![](https://paper-attachments.dropboxusercontent.com/s_AF10D3C89180AF3852A305EF4DE5BA2E2317FBE4BD9DA7F6A7B3728E3E789046_1705699082262_imagen.png)


La de sesión

![](https://paper-attachments.dropboxusercontent.com/s_AF10D3C89180AF3852A305EF4DE5BA2E2317FBE4BD9DA7F6A7B3728E3E789046_1705699101385_imagen.png)


