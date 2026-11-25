# Notas: API de Autenticación de Pocket

- Todas las peticiones HTTPS y POST.
- Necesito un **web server** para poder **recibir el callback** de Pocket.
- Son dos pasos:
    - Obtener un *response token* (code)
    - Usar el *response token* para obtener un **Pocket token**
        - Con este es que se pueden hacer peticiones al API
# Petición para obtener code
    POST /v3/oauth/request HTTP/1.1
    Host: getpocket.com
    Content-Type: application/json; charset=UTF-8
    X-Accept: application/json
    
    {
      "consumer_key":"KEY",
      "redirect_uri":"https://poffersyncer.herokuapp.com/letsgo"
    }

Respuesta

    HTTP/1.1 200 OK
    Content-Type: application/json
    Status: 200 OK
    
    {"code":"dcba4321-dcba-4321-dcba-4321dc"}

Ese código es el que servirá para:

- Autorizar el acceso de la aplicación a la cuenta del usuario
- y obtener un **Pocket token**.
## Sobre los errores en las respuestas

Todo lo que sea 200 está bien, sino hubo error.

> Tener en cuenta que los errores tienen HTTP Status Code, X-Error-Code y X-Error (el texto del error).
# Redireccionar a Pocket para Autorizar

Este paso lo que va a hacer es abrir una página de inicio de sesión y luego de autorización para que la aplicación (Poffer Syncer) tenga acceso a la cuenta del usuario.

Para web hay que enviar al usuario a la página de Pocket

    https://getpocket.com/auth/authorize?request_token=YOUR_REQUEST_TOKEN&redirect_uri=YOUR_REDIRECT_URI

Así:

    https://getpocket.com/auth/authorize?request_token=dcba4321-dcba-4321-dcba-4321dc&redirect_uri=https://poffersyncer.herokuapp.com/letsgo

Cuando el usuario acepte o rechace la aplicación, se redirigirá a la página indicada en `redirect_uri`.

# Finalmente, Obtenemos el Pocket Token

Hay que convertir el *response token* del paso inicial a un **pocket token**.

Para obtenerlo, hay que hacer una petición a:

    POST /v3/oauth/authorize HTTP/1.1
    Host: getpocket.com
    Content-Type: application/json; charset=UTF-8
    X-Accept: application/json
    
    {
      "consumer_key":"1234-abcd1234abcd1234abcd1234",
      "code":"dcba4321-dcba-4321-dcba-4321dc"
    }

que responderá con:

    HTTP/1.1 200 OK
    Content-Type: application/json
    Status: 200 OK
    
    {
      "access_token":"KEY",
      "username":"pocketuser"
    }


> Tener en cuenta que los errores tienen HTTP Status Code, X-Error-Code y X-Error (el texto del error).

Ya con este paso y el `access_token` de nuestro lado, podemos hacer peticiones a los endpoints de Retrieve y Modify.

🥳 

