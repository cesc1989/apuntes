# Plan Ciclo 4 ⏳ 
Ya tenemos algo bien armado con respecto a las gemas. Entendemos el flujo para hacer peticiones a Pocket y a Hootsuite y tenemos el esqueleto de la aplicación Rails.

Ahora, necesitamos poder integrar las gemas en el proyecto Rails. Para esto, las gemas deben poder recibir el access token desde una fuente externa o desde variable de entorno. El proyecto Rails también debe poder guardar estos tokens.

# Estado de las cosas
## Recibir `access_token` en gema Pocket

Para esto hay que hacer cambio de lectura del access token.

Ir de esto:

    # lib/pocket/articles.rb
    
      REQUEST_PARAMS = {
          count: 10,
          tag: "schedule",
          consumer_key: ENV['POCKET_CONSUMER_KEY'],
          access_token: ENV['ACCESS_TOKEN']
        }
    
        def initialize
          @client = Client.new
        end
    
        def articles
          r = @client.call(endpoint: RETRIEVE_ENDPOINT, params: REQUEST_PARAMS)
    

A esto:

    # lib/pocket/articles.rb
    
      REQUEST_PARAMS = {
          count: 10,
          tag: "schedule"
        }
    
        def initialize(args)
          @client = Client.new
          @keys = {
            access_token: args.fetch(:access_token),
            consumer_key: args.fetch(:consumer_key)
          }
        end
    
        def articles
          r = @client.call(endpoint: RETRIEVE_ENDPOINT, params: REQUEST_PARAMS.merge(@keys)
    


## Recibir `access_token` en gema Hootsuite

Para esto hay que hacer cambio de lectura del access token.

Pasar de esto:

    # lib/hootsuite.rb
    
    module Hootsuite
      class Error < StandardError; end
    
      class Client
        BASE_URL = 'https://platform.hootsuite.com'
        ACCESS_TOKEN = ENV['HOOTSUITE_ACCESS_TOKEN']
    
        def get(endpoint)
          HTTP.auth("Bearer #{ACCESS_TOKEN}")
              .get("#{BASE_URL}/#{endpoint}")
        end
    
        def post(endpoint, params)
          HTTP.auth("Bearer #{ACCESS_TOKEN}")
              .post("#{BASE_URL}/#{endpoint}", json: params)
        end

A algo como esto:

    # lib/hootsuite.rb
    
    module Hootsuite
      class Error < StandardError; end
    
      class Client
        BASE_URL = 'https://platform.hootsuite.com'
    
        def initialize(args)
          @access_token = args.fetch(:access_token, ENV['HOOTSUITE_ACCESS_TOKEN'])
        end
    
        def get(endpoint)
          HTTP.auth("Bearer #{@access_token}")
              .get("#{BASE_URL}/#{endpoint}")
        end
    
        def post(endpoint, params)
          HTTP.auth("Bearer #{@access_token}")
              .post("#{BASE_URL}/#{endpoint}", json: params)
        end


## Preparar el proyecto Rails para almacenar los tokens

Ya tenemos los modelos `HootsuiteToken` y `PocketToken` para esto. Solo necesitan ajustar las validaciones y probar.

# Lo que hay que hacer
- Leer pocket token desde fuente externa
- Leer hootsuite token desde fuente externa
- Validaciones e integrar las gemas al proyecto web
- Correr rake task de prueba

