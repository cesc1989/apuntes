# Nombrar bien las cosas y su importancia
En los últimos días me he encontrado con diferentes instancias de personas haciendo énfasis en nombras bien las cosas.

## Conclusiones

Debo sacar al menos 5-10 minutos para pensar el nombre que le daré a clases u objetos. Esto pensando en que sean claros y expresen bien la idea de su existencia


## Algunos ejemplos
- Email de Jason Swett - [Ir abajo](https://paper.dropbox.com/doc/Nombrar-bien-las-cosas-y-su-importancia--CJXUeSccSbs2MRDQ8kxiOb8_Ag-2YAZ9IyrG9ZIjoYJFplld#:uid=417602911996963950826099&h2=Email-de-Jason-Swett:-Naming).
- Email de Robby Russell - [Ir abajo](https://paper.dropbox.com/doc/Nombrar-bien-las-cosas-y-su-importancia--CJXUeSccSbs2MRDQ8kxiOb8_Ag-2YAZ9IyrG9ZIjoYJFplld#:uid=916285576455399552001543&h2=Email-de-Robby-Russell:-The-th).
- Riffing on Rails: First Ruby Friend
- Veces que Ryan corrigió nombrar las cosas en los PRs

Veamos en detalle algunos de estos ejemplos.

# Email de Jason Swett: Naming
> There are concepts in the application code, sometimes quite fundamental ones, that aren't named quite right. So every time you X in the codebase, you have to think to yourself, "Oh yeah, X doesn't really mean X, it means Y."

En Edge: CarePlan es un Episode.


> The lesson we should take away from this is that **bad names incur a dear cost**, and so **we shouldn't be afraid to pay a high price for a good name**.

Al nombrar, estamos es decidiendo y expresando lo que cada cosa es.

> One last note about naming: it's not really naming.
> 
> We're not really deciding what the thing is called. We're deciding what the thing IS! Big difference.

Finalmente:

> So a statement like "naming is important" is actually pretty wrong. It's more like "conceptual modeling is important". But that's wrong too. It's really more like "an information system is made out of conceptual models".
> 
> Time spent getting the conceptual model right is always time well-invested.


# Email de Robby Russell: The things we name

Al inicio menciona un “Glossary” con términos que no son claros cuando el cliente los menciona ya que no existen en el repositorio.

Ejemplo de un glosario lo vi en Luna en Confluence. Hay un montón de términos ahí definidos.


> Nota: creo que debo hacer un glosario para los proyectos de Luna.

Esto es lo que pasa normalmente:

> The underlying problem here is that when **things were named during the early phases of the project, they didn’t reflect the language that was being used by the organization** that was having it built. Rather, the original developers took the requests and attempted to use generic names for some models and/or a bunch of acronyms.

Para resolver esto, Robby sugiere usar y practicar el [*Ubiquitous language*](https://www.martinfowler.com/bliki/UbiquitousLanguage.html)  “Lenguaje Ubicuo” ([RAE](https://dle.rae.es/ubicuo)). Es decir, expresar en código el lenguaje y terminología que usa el cliente del proyecto.

Robby da varios ejemplos:

> Does your travel agency have **users** that need to login and conduct travel agency work or do you employ travel **agents** that need to login and conduct their work?


> Does your toolbox have a collection of **items** or does your toolbox have a collection of **tools**?


> Does your photo gallery allow users to upload **images** or should they be uploading **photos**?

Este es interesante:

> Do non-profits send **applications** to your foundation or do they submit **grant applications**? (Also, be wary of naming things 'Applications' in your applications. It can get tricky!)


# Riffing on Rails: First Ruby Friend

En esa sesión pude ver como el enfoque práctico y simplificado para sacar la idea de lo que querían hacer les permitía dejar tiempo para buscar y pensar lo que significaba cada clase.

Código de la sesión: https://github.com/kaspth/riffing-on-rails/blob/main/friend.rb

