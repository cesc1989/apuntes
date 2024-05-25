# Hypermedia-Friendly Scripting - Apuntes
Este ensayo explica cómo sería una app con scripts Hypermedia-Friendly o hasta qué punto son aceptables bajo este enfoque.

The basic rules for hypermedia-friendly scripting are:

- Respect HATEOAS
- Client-side only state is OK
- Use events to communicate between components
- Use islands to isolate non-hypermedia components from the rest of your application
- Optionally, consider inline scripting
## The Prime Directive
> **Practically, this means that scripting should avoid making non-hypermedia exchanges over the network with a server.**
> 
> (…) hypermedia-friendly scripting should avoid the use of `fetch()` and `XMLHttpRequest` unless the responses from the server use a hypermedia of some sort (e.g. HTML), rather than a data API format (e.g. plain JSON).

En resumen, para que el scripting respete a HATEOAS:

> The idea is to use scripting to improve the hypermedia experience by providing features and functionality that are not part of the standard hypermedia (HTML) tool set, but do so in a way that plays well with HTML, rather than relegating HTML to a mere UI description language within a larger JavaScript application, as many SPA frameworks do.


## State

Estado solo en frontend está bien siempre que no sea sincronizado al backend. Si se va a sincronizar con backend, debe seguir las reglas de Hypermedia.

> ephemeral client-side state is fine in a Hypermedia-Driven Application, because the state is purely front-end. No system state is being updated with this sort of scripting.

En definitiva:

> The crucial aspect to consider is whether any state updated on the client side needs to be synchronized with the server.
> If yes, then a hypermedia exchange should be used. If no, then it is fine to keep the state client-side only.


## Events


## Islands

Esto es una noción reciente en el mundo frontend:

> The [islands architecture](https://www.patterns.dev/vanilla/islands-architecture) encourages small, focused chunks of interactivity within server-rendered web pages.

¿cómo vienen a jugar en el mundo Hypermedia?

> the most hypermedia-friendly approach is to use the island architecture. This isolates non-hypermedia components from the rest of the Hypermedia-Driven Application.


> it is typically easier to embed non-hypermedia islands within a larger Hypermedia-Driven Application, rather than vice-versa.
> 
> -- Deniz Akşimşek

Sobre lo anterior me recuerda mucho cuando leí sobre React y cómo era usado, inicialmente en Facebook: dar mayor interactividad en partes específicas de la plataforma.


## Inline Scripts

Esta regla dice que en vez de poner scripts en archivos diferentes, el scripting debe hacerse directo en el archivo HTML. O sea, scripts inline.

Como Tailwind para CSS. Es algo descabellado pero tiene sus consideraciones al respecto en el ensayo.

La clave y por lo cual ambos enfoques son válidos es que tanto inline scripting como archivos de componentes es que satisfacen “Locality of Behavior”. Todo lo que hace el script queda ubicado en un mismo espacio.

Eso sí:

> with inline scripts, there should be a soft limit to the amount of scripting done directly within the hypermedia. You don’t want to overwhelm your hypermedia with scripting, so that it becomes difficult to understand “the shape” of the hypermedia document.


## Pragmatismo

Al final, hay que hacer el trabajo.

Mientras sea posible, envuelve el scripting en una forma que quede aislado de Hypermedia pero si no es posible no te malgastes en mantener la pureza de Hypermedia.

