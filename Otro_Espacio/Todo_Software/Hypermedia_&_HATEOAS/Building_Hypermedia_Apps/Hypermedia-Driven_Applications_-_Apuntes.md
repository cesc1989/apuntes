# Hypermedia-Driven Applications - Apuntes
La conclusión principal de este ensayo es:

> The **Hypermedia Driven Application (HDA)** architecture is a new/old approach to building web applications. It combines the simplicity & flexibility of traditional Multi-Page Applications (MPAs) with the better user experience of [Single-Page Applications](https://en.wikipedia.org/wiki/Single-page_application) (SPAs).

Crear aplicaciones que aprovechen las bondades de Hypermedia y el enfoque tradicional pero dotándoles de interactividad usando técnicas modernas de Frontend usando técnicas declarativas.


- An HDA uses *declarative, HTML-embedded syntax* rather than imperative scripting to achieve better front-end interactivity
- An HDA interacts with the server **in terms of hypermedia** (i.e. HTML) rather than a non-hypermedia format (e.g. JSON)

Esto que menciona es ejemplificado en lo que hace Hotwire. Hacer la petición estilo AJAX, el servidor devuelve HTML y reemplazar el HTML nuevo sin recargar la página.

Cuando se necesite mayor interactividad, se usaría StimulusJS.

> Scripting is used mainly to enhance the front-end experience of the application

