# Cómo usar las gemas
Aquí describo cómo usar cada gema de manera independiente.

# Gema Pocket

Para más detalles ver [+Notas: API de Autenticación de Pocket](https://paper.dropbox.com/doc/Notas-API-de-Autenticacion-de-Pocket-8lSTZ83w1ZBjg4xVquqzf) .

Estando en la carpeta del proyecto y luego de haber ejecutado el comando `bundle install` y haber configurado las variables de entorno con lo que necesita la integración a Pocket.

    POCKET_CONSUMER_KEY=96566-1a889f8c01e2ae59c4b6619d
    CALLBACK_URL="http://localhost:4567/oauth/callback"

Luego de eso se ejecuta el servidor web local del proyecto:

    $ ruby local_server.rb

Y luego ir a `localhost:4567`.

## Conectando con Pocket

Hay que conectarse mediante OAuth para poder obtener el access token con el cual hacer las peticiones a la API.

Se sigue el enlace y en la página de Pocket se inicia sesión y se acepta la petición para dar permisos. Cuando sea todo correcto aparecerá el Pocket Token que se necesita para hacer peticiones.

![](https://paper-attachments.dropbox.com/s_9A9CE8B5CAEB4F389D30644B2F0E86DFD038929579DF6A1A883A65E0827151B7_1664573883150_image.png)



## ¿Qué pasa tras bambalinas?

Cuando se da clic en el enlace “Connect with Pocket” se está yendo a la ruta definida en:

    get "/oauth/connect" do
      # ...
    end

En ese punto se instancia a `Pocket::Oauth::Connect` para hacer una petición y pedir un código (response token) el cual deberá convertirse en pocket token.

En este paso se ejecuta:

    code = connect_client.connect
    session[:code] = code
    puts code

Ese código se lleva a autorizar en la página de Pocket dónde se aceptan los permisos. 

Cuando se acepta y se vuelve a la página local, se llega a la página indicada por `CALLBACK_URL`. En nuestro caso será:

    get "/oauth/callback" do
      # ...
    end

Allí nuevamente se instancia a `Pocket::Oauth::Connect` pero esta vez se envía el mensaje para obtener el Pocket Token final:

    pocket_token = Pocket::Oauth::Connect.new.pocket_token(session[:code])
    session[:access_token] = pocket_token
    puts pocket_token

Y somos redirigidos a la página inicial.

Con dicho Pocket Token es que podemos hacer peticiones al API.

Para usar dicho Pocket Token en la gema estando en una consola de IRB, habría que configurarla como variable de entorno:

    POCKET_CONSUMER_KEY=KEY
    CALLBACK_URL="http://localhost:4567/oauth/callback"
    ACCESS_TOKEN=TOKEN

Y luego usar la gema:

    $ bundle exec ./bin/console
    
    001> ap Pocket::Articles.new.articles
    [
        [0] #<Pocket::Articles::Article:0x00007faa9fa6e568 @article={"item_id"=>"970551832", "resolved_id"=>"970551832", "given_url"=>"https://blog.jaibot.com/the-copenhagen-interpretation-of-ethics/", "given_title"=>"The Copenhagen Interpretation of Ethics – Almost No One is Evil. Almost Eve", "favorite"=>"1", "status"=>"0", "time_added"=>"1634482759", "time_updated"=>"1664574705", "time_read"=>"0", "time_favorited"=>"1652239676", "sort_id"=>0, "resolved_title"=>"The Copenhagen Interpretation of Ethics", "resolved_url"=>"https://blog.jaibot.com/the-copenhagen-interpretation-of-ethics/", "excerpt"=>"The Copenhagen Interpretation of quantum mechanics says that you can have a particle spinning clockwise and counterclockwise at the same time – until you look at it, at which point it definitely becomes one or the other. The theory claims that observing reality fundamentally changes it.", "is_article"=>"1", "is_index"=>"0", "has_video"=>"0", "has_image"=>"0", "word_count"=>"1418", "lang"=>"en", "time_to_read"=>6, "top_image_url"=>"https://blog.jaibot.com/wp-content/uploads/2017/12/EA_logo_twitter_bred.png", "listen_duration_estimate"=>549}>
    ]
     => nil


# Gema Hootsuite

Para más detalles ver [+Notas: API de Autenticación de Hootsuite](https://paper.dropbox.com/doc/Notas-API-de-Autenticacion-de-Hootsuite-a7yZbA1Poe2wDL0gDv8hN) .

Estando en la carpeta del proyecto y luego de haber ejecutado el comando `bundle install` y haber configurado las variables de entorno con lo que necesita la integración a Hootsuite.

    CLIENT_ID="ID"
    SECRET_ID="SECRET"
    REDIRECT_URI="http://localhost:4567/oauth/callback"

se ejecuta el servidor web local del proyecto:

    $ ruby local_server.rb

Y después ir a `localhost:4567`.

## Conectando con Hootsuite

Hay que conectarse mediante OAuth para poder obtener el access token con el cual hacer las peticiones a la API.

Se sigue el enlace y en la página de Hootsuite se inicia sesión y se acepta la petición para dar permisos. Cuando sea todo correcto aparecerán varios valores incluyendo el token que se necesita para hacer peticiones.

![](https://paper-attachments.dropbox.com/s_9A9CE8B5CAEB4F389D30644B2F0E86DFD038929579DF6A1A883A65E0827151B7_1664831469513_image.png)

## ¿Qué pasa tras bambalinas?

Cuando se da clic en el enlace “Connect with Hootsuite” se está yendo a una URL que tiene varios de los valores configurados en el entorno.

    get "/" do
      if session[:access_token]
        # ...
      else
        "<a href=#{Hootsuite::Oauth::Authenticate.auth_url}>Connect with Hootsuite</a>"
      end
    end

Cuando se aceptan los permisos en la página de Hootsuite, volvemos a la `REDIRECT_URI` y ahí se usa el código que nos devuelve Hootsuite para pedir el access token y el refresh token:

    get "/oauth/callback" do
      code = request.params['code']
    
      response = Hootsuite::Oauth::Authenticate.new(code).request
      
      # ...
    end

Con el access token que devuelve se pueden empezar a hacer peticiones al API de Hootsuite.

Una vez tenemos el access token, lo configuramos como variable de entorno:

    # .env
    HOOTSUITE_ACCESS_TOKEN="vEPnR0a88M1QAzRC5-eHH86PSoRQ70Id1YAkker4TpU.LVa616Fycrlu-8EmdJNvXjOW22OBP3y9SIVCoUWRZl0"

y podemos probar la gema

    $ bundle exec ./bin/console
    
    001 > pp Hootsuite::SocialProfile.new.social_profile_ids
    ["134615263", "134615271"]

