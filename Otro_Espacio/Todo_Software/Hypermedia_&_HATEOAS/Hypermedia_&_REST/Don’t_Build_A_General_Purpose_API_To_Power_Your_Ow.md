# Don’t Build A General Purpose API To Power Your Own Front End - Apuntes
Artículo original https://max.engineer/server-informed-ui

El consenso es que no se necesita una API única para nutrir al frontend. Necesitas dos APIs independientes.

# Por qué no?

When you design a general purpose API, you have to figure out a bunch of annoying stuff:


- How to optimize requests to multiple endpoints
- How to weigh the benefit of new features against the cost of new API requests

Son ejemplos.

Caso en Luna en Dashboard que tocó paralelizar las peticiones porque todo estaba lento.

O en Bucket cada que tocaba pensar en un nuevo endpoint por un cambio de diseño.

# So what do you suggest?

Trata el frontend como la otra mitad de la aplicación:

> Imagine if you could just send it the whole “page” worth of JSON. Make an endpoint for `/page/a` and render the whole JSON for `/page/a` there. Do this for every page. Don’t force your front-end developers to send a bunch of individual requests to render a complex page. Stop annoying them with contrived limitations.


> And in that JSON, actually render the page. Don’t render abstract models and collections. Render concrete boxes, sections, paragraphs, lists. Render the visual page structure.


    {
      "section1": {
        "topBoxTitle": "Foo",
        "leftBoxTitle": "Bar",
        "linkToClose": "https://…"
      },
      "section2": {
        …
      }
    }

¿Es acaso esto lo que está viendo Ryan al pedirme Mobile Progress Forms API y la nueva para los Forms?

# How is that better exactly?
> No more **“what API workflows do we need to introduce to sort of make this page possible almost? 🤔”**. You can keep “page a” dumb to only do what it needs to do. You test the crap out of “page a” for bugs, security, performance. You can even fetch everything for “page a” in a single big SQL query. You can cache the entire JSON payload of “page a”.


> When a stakeholder tells you to change “page a” you will be able to literally go ahead and change “page a”, instead of spending meetings figuring out how your backend API could accommodate the change in “page a”. It’s not a choreographed conglomeration of API requests.
# Opiniones en Contra
## But I want my front-end team to have freedom! (Or, I want my front-end to be decoupled!)


> frontend doesn’t really have freedom. When they send you 7 requests to render a single page, that’s not freedom.

Es solo darle vueltas al tema para lograr un requerimiento básico.

## But we actually want a general purpose API anyway, so this is 2 birds with 1 stone, no?

La API pública debe ser para flujos de servicios externos.

La API privada debe ser para el frontend del producto. Así más fácil cambiar cada que el product manager le antoje.

## But we can reuse this API for the mobile app too!

Haz otra API privada. Las apps móviles tienen diferentes estructuras y flujos. Tiene más sentido hacer una API privada acorde para cumplir todo eso en pocas peticiones.

Esto es justo lo que pasa en Luna y la v3 del API que es para Mobile y ahora lo que estoy haciendo yo que V1 que es para web.

Y también cuando hice Mobile Progress Forms API.

