# Buscando un gestor de marcadores (Bookmarks)

## Conclusiones



# Las Alternativas

> [!important]
> La gran diferencia entre un gestor de marcadores y un _read it later_ es que el primero es solo guardar los enlaces en carpetas. Como lo hacen los navegadores web.
> 
> En cambio el segundo guarda el texto y permite leerlo en una interfaz más limpia, además de permitir resaltos o anotaciones.
> 
> En caso de un gestor de marcadores que busco es solo guardar enlaces en carpetas/sub carpetas. Para leer después ya estoy decidiendo entre Omnivore, Readeck o Wallabag en [[Buscando un gestor de Marcadores (Bookmarks)]]

A la fecha, todos mis marcadores estén en [Raindrop](https://raindrop.io/). Un buen servicio para guardar y organizar los marcadores. Ya lleva bastante tiempo en el mercado. Lo conozco desde 2016 y creo que existe desde antes.

Raindrop me gusta y me sirve pero, a mí parecer, hace cosas de más. Raindrop como es ahora mismo no solo gestiona marcadores sino que también permite leer las webs y tomar apuntes en forma de resaltos o notas. **Eso me parece de más** y hace que confunda el uso que quiera darle a la herramienta.

![[50.raindrop.bookmarks.and.annotations.png]]

_Esta es una vista de Raindrop. Se puede ver una lista de carpetas en la barra izquierda y en la derecha un artículo que se puede visualizar y anotar. Demasiado cargado._

La oferta de Raindrop está muy bien pero **no es lo que yo busco en este tipo de programas**. Es por eso que estoy buscando alternativas.

## ¿Qué busco en un gestor de marcadores?

Creo que en este orden busco:

1. Guardar enlaces: solo necesito el mínimo de información como título, url, etiquetas.
2. Organizar marcadores en carpetas: que se puedan anidar en varios niveles.

Lo demás es añadidura. Creo que de los plus el más deseable es la posibilidad de archivar el contenido del sitio web enlazado. Pasa mucho que tengo marcadores viejisimos y cuando los voy a abrir el sitio ya murió y con él la información.

Poder archivar ese contenido sería invaluable para mí.

> [!info]
> Archivar es una característica de Raindrop en su versión Pro.

## ¿Qué alternativas he encontrado?

- [Linkwarden](https://github.com/linkwarden/linkwarden)
- [Linkding](https://linkding.link/)



# Linkding

> [!info]
> Sitio web: https://linkding.link/
> Escrito en Python (Django + Svelte para la UI). Repo -> https://github.com/sissbruecker/linkding

Esta es la interfaz de Linkding. Algo así de sencillo me gusta muchísimo.

![[51.linkding.ui.png]]

Linkding lo puedo probar y usar en Pikapods. Sin embargo, hay una pega, Linkding no soporta la creación de colecciones o carpetas. Hay un issue en el repositorio pero el principal mantenedor no quiere hacerlo.

Así que si quiero usar este programa tendría que encontrar la forma de organizar usando las etíquetas.

# Linkwarden

> [!info]
> Sitio web: https://linkwarden.app/
> Escrito en TypeScript (NextJS + Tailwind). Repo -> https://github.com/linkwarden/linkwarden

> [!note]
> Linkwarden también es self hosted pero no hay oferta de este programa en Pikapods. Si quiero usarlo me toca en su nube que ofrece prueba de 14 días y luego son 150K/año.

Este sí tiene carpetas anidadas pero la UI me parece más cargada que en Linkding. De resto, bien. Pude importar los marcadores del export que generó Raindop (o sea que hay buena compatibilidad entre servicios).

> [!warning]
> El export total de datos lo hace en un archivo JSON.
> Esto no es muy deseable. Hay una [petición en los issues](https://github.com/linkwarden/linkwarden/issues/587) pero a la fecha (Oct. 2024) no hay implementación.
>
> Como workaround, un usuario creó un [script para convertir](https://gist.github.com/arnavpraneet/9798d6f2d33913a544dddbbc90f3df2e) el JSON a HTML Netscape.