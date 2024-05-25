# HATEOAS Is For Humans - Apuntes
HATEOAS como concepto es sencillo: Un servidor envía los datos y las operaciones de la red en la respuesta a la petición del cliente. El cliente no tiene ningún conocimiento especial sobre esa data ni operaciones de red.


## ¿Qué es HATEOAS?

Sus siglas:

Hypermedia As The Engine Of Application State.

**Significado**
Todo el estado (datos y acciones de red para efectuar en los datos) es devuelto por el servidor en Hypermedia (HTML).

El cliente no sabe nada sobre los datos ni las operaciones. Todo eso es devuelto por el servidor unicamente.


----------

Es aquí donde brilla el navegador web cuando se le entrega HTML.


----------


# HATEOAS es para humanos

Esto en el sentido de que si una aplicación va a retornar Hypermedia es para que sea recibida por un cliente que entienda Hypermedia, es decir, navegadores web.

Unas aplicación que retorne Hypermedia no debe enviarlo a un cliente que no lo entienda porque será frustrante y más difícil de trabajar.

Y algo clave es:


> And that gets to crux of the issue: the code doesn’t (yet) have [agency](https://en.wikipedia.org/wiki/Agency_(philosophy)).
> It can’t reasonably decide what to do in the face of new and unexpected actions.

Agency o [Agencia](https://es.wikipedia.org/wiki/Agencia_(filosof%C3%ADa)) es la capacidad de actuar en un sistema.

Un cliente, digamos, una SPA no tiene agencia sobre una respuesta Hypermedia porque no sabe qué hacer con eso.

En cambio una persona, que usando un navegador web, ve una respuesta en Hypermedia, podrá ejercer agencia sobre esa respuesta y darle uso y sentido.


> a system satisfying HATEOAS is wasted if the hypermedia isn’t being consumed by something with agency

