# HATEOAS - Apuntes
HATEOAS es

Hypermedia
As
The
Engine
Of
Application
State

Al cumplir esta restricción de REST, el cliente interactúa con el servidor que devuelve información dinámica mediante el uso de Hypermedia.

[Hypermedia](https://en.wikipedia.org/wiki/Hypermedia) es medio de información no linear el cual incluye gráficos, vídeo, audio, texto y enlaces.

## Usando Hypermedia

Un ejemplo de un endpoint que responde rigiéndose por Hypermedia:

    GET /accounts/12345 HTTP/1.1
    Host: bank.example.com
    
    HTTP/1.1 200 OK
    
    <html>
      <body>
        <div>Account number: 12345</div>
        <div>Balance: $100.00 USD</div>
        <div>Links:
            <a href="/accounts/12345/deposits">deposits</a>
            <a href="/accounts/12345/withdrawals">withdrawals</a>
            <a href="/accounts/12345/transfers">transfers</a>
            <a href="/accounts/12345/close-requests">close-requests</a>
        </div>
      <body>
    </html>


> The web browser does not know about the concept of an overdrawn account or, indeed, even what an account is. It simply knows how to present hypermedia representations to a user.

Esto es la definición de Hypermedia. El cliente no necesita saber más nada. El servidor (el modelo) sabe toda la lógica. El cliente solo le debe importar la presentación de los datos.


> the notion of the Hypermedia being the Engine of Application State. What actions are possible varies as the state of the resource varies and this information is encoded in the hypermedia


## Respuesta JSON

El contraste queda claro al ver una respuesta JSON donde el cliente tiene que saber mucho sobre la lógica y el estado:

    HTTP/1.1 200 OK
    
    {
        "account": {
            "account_number": 12345,
            "balance": {
                "currency": "usd",
                "value": -50.00
            },
            "status": "overdrawn"
        }
    }


> The client must also know what URLs must be used for manipulation of this resource since they are not encoded in the response. This would typically be achieved by consulting documentation for the JSON API.

Y las diferencias son más claras en el sentido que:

> in the RESTful, HATEOAS HTML representation, all operations are encoded directly in the response. In the JSON API example, out-of-band information is necessary for processing and working with the remote resource.

Las acciones posibles son devueltas por el servidor de la aplicación. Este tiene mejor conocimiento de la lógica envuelta.

En cambio, en el JSON se necesita consultar documentación y el cliente debe saber sobre las posibles acciones.

# Conclusiones

Una API que retorne hypermedia simplifica el trabajo del cliente ya que la lógica manejada por el servidor puede expresarse mejor en la respuesta. Una respuesta HTML puede incluir todas las acciones posibles por el cliente (en la forma de enlaces y formularios).

La respuesta es auto-conclusa: no se necesita revisar documentación ni entender lógica por aparte porque se incluye todo lo necesario para mostrar datos y que el usuario pueda tomar acciones.

