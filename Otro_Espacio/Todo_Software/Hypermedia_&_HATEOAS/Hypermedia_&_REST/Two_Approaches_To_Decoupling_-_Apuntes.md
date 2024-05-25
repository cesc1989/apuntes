# Two Approaches To Decoupling - Apuntes
In this essay we will look at two different types of decoupling in the context of web applications:

- Decoupling at the *application level* via a generic JSON Data API
- Decoupling at the *network architecture level* via a hypermedia API
## Acoplamiento / Desacoplamiento
> Coupling is a property of a software system in which two modules or aspects of the system have a high degree of interdependence. Decoupling software is the act of reducing this interdependence between unrelated modules so that they can evolve independently of one another.


# Desacoplamiento mediante una API JSON

Que los endpoints de un API devuelva valores muy ligados a una vista es contraproducente y hace que haya mucho acoplamiento entre el cliente y el API.

Lo ideal es que haya un API más genérica y el cliente use lo que necesite. 


> The worst part of my job these days is designing APIs for front-end developers. The conversation goes inevitably as:
> 
> Dev – So, this screen has data element x,y,z… could you please create an API with the response format {x: , y:, z: }
> 
> Me – Ok

These sorts of requests end up recoupling the front-end and back-end code: the JSON API is no longer providing a generic JSON Data API, but rather a specific API for the front-end needs.


> Worse, these front-end needs will often change frequently as your application evolves, necessitating the modification of your JSON API.

Y esto pasa en Luna (en Luxe):

> This problem leads to the “versioning hell” that many JSON Data API developers face when supporting both web applications as well as other non-web application clients.

Por eso hay v1, v2, v3 cada uno con internal y external.

## Solución: una API de datos genérica y una API de aplicación

build two JSON APIs:

- An application specific JSON API that can be modified as needed
- A general purpose JSON API that can be consumed by other clients such as mobile, etc.
# Desacoplamiento mediante Hypermedia

Tomemos este ejemplo:

    https://example.com/account/12345 that we saw above:
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

y queremos cambiar el API (sí lo es solo que devuelve HTML) para que no se hagan transfers ni close-requests, la respuesta sería así:

    HTTP/1.1 200 OK
    
    <html>
      <body>
        <div>Account number: 12345</div>
        <div>Balance: $100.00 USD</div>
        <div>Links:
            <a href="/accounts/12345/deposits">deposits</a>
            <a href="/accounts/12345/withdrawals">withdrawals</a>
        </div>
      <body>
    </html>

Se quitaron los enlaces de la respuesta así que ya ningún usuario vería esos enlaces y no usarían la versión “anterior” de la API.

Esto trae el beneficio de poder cambiar la API (Hypermedia) con más flexibilidad sin miedo a romper los clientes porque la lógica va toda devuelta en la respuesta.

## Aclaración

Este tipo de respuestas HTML solo tienen sentidos en sistemas donde se aproveche HTML a pleno. No tienen ningún sentido parsear HTML para extraer datos.

Eso no es de ningún beneficio.


> You can take advantage of the flexibility of hypermedia for your own web application, while providing a general purpose JSON API for mobile applications, third party applications, etc.



