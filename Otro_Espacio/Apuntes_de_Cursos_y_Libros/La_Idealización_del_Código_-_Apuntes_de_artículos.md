# La Idealización del Código - Apuntes de artículos
Estos son apuntes de los artículos inspiración para el artículo.

Artículo publicado → https://otroespacioblog.wordpress.com/2021/02/25/la-idealizacion-del-codigo/

Artículos:

- [Clean, DRY, SOLID Spaghetti](https://dev.to/codemouse92/clean-dry-solid-spaghetti-1lgm)
- [You don’t believe in Clean Code](https://dev.to/danlebrero/you-dont-believe-in-clean-code-113n)
- [Are There Actually Companies out There That Write Good Code?](https://dev.to/daedtech/are-there-actually-companies-out-there-that-write-good-code-1pbo)


# Clean, DRY, SOLID Spaghetti

A pesar de todos los esfuerzos, la base de código aún podría ser terrible.

Los principios tienen su valor en todo caso. Documentación, estándares y estilo son importantes.


> Regardless of purpose, language, or methodology, ALL software must fulfill exactly two criteria:
> 1) The software must accomplish its stated goals.
> 2) The software must be maintainable (thus, readable) by future developers.

El problema es que para la primera el código puede estar en cualquier forma. Se deja mucho la segunda como algo para luego.

## Siguiendo los principios pero con moderación

Los principios ayudan pero si se abusan son peor que la causa. Hay que buscar un balance.

Un poco de repetición no tiene nada de malo. No todo tiene que DRYearse.

## Sobre comentarios

Hay que dejar comentarios clarificadores y que expliquen el porqué de todo aquello que se pueda prestar para confusión. Se podría seguir el principio de “[Commenting Showing Intent](https://standards.mousepawmedia.com/en/latest/csi.html)”.

Interesante este principio. Ver la [sección de Tono](https://standards.mousepawmedia.com/en/latest/csi.html#tone).

----------

No abusar de SOLID al extremo ni de prácticas de Clean arquitecture.

Los tests son para verificar que el código funcione y deben estar escritos a consciencia y con el firme propósito de ayudar a la calidad y a futuros refactors.

Tampoco hay que abusar de guías de estilo. Todo necesita moderación.

## De los comentarios
> Sometimes the benefits exceed the costs, sometimes they don't. In my experience, great programmers are good at doing that math, but bad programmers don't even realize that there is math to be done.

Del libro “Effective Project Management” (Robert K. Wysocki) que se puede aplicar a programación:

> Projects are unique, and each one is different from all others that have preceded it. That uniqueness requires a unique approach that continually adapts as new characteristics of the project emerge.

