# How Did REST Come To Mean The Opposite of REST? - Apuntes

# Conclusiones

Decir que una API que retorna JSON es RESTful está errado.

Para que una API sea RESTful sus respuestas deben ser auto-descritas. Es decir, la respuesta contiene todo lo necesario para mostrar la lógica pertinente, el estado del cliente y además las posibles acciones para los usuarios del navegador.

# Apuntes
> at the time of his writing (1999-2000), there were no JSON APIs: he was describing the web as it existed at that time, with HTML being exchanged over HTTP


> REST described a network architecture, and it was defined in terms of constraints on an API, constraints that needed to be met in order to be considered a RESTful API.


## Ejemplos de respuestas

Veamos una HTML:

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

vs una JSON

    HTTP/1.1 200 OK
    
    {
        "account_number": 12345,
        "balance": {
            "currency": "usd",
            "value": 100.00
         },
         "status": "good"
    }

Captura para mostrar todos los párrafos explicativos juntos

![](https://paper-attachments.dropboxusercontent.com/s_6F9C0916D6A7F7394280A26616CA4941119819B43ED02FD49735987C071A42B6_1699931247706_imagen.png)


La respuesta JSON:

> The JSON response is not self-describing and does not encode the state of the resource within a hypermedia. It therefore fails the uniform interface constraint of REST, and, thus, is not RESTful.

Un API JSON no es RESTful porque no es auto-descriptiva. Le deja el trabajo de entender el estado al cliente cuando el mismo servidor podría saberlo (y devolver HTML).

En definitiva un sistema RESTful correcto:
in a RESTful system, you should be able to enter the system through a single URL and, from that point on, all navigation and actions taken within the system should be entirely provided through self-describing hypermedia.


## GraphQL no es RESTful tampoco
> GraphQL couldn’t be less RESTful: you absolutely have to have documentation to understand how to work with an API that uses GraphQL.


