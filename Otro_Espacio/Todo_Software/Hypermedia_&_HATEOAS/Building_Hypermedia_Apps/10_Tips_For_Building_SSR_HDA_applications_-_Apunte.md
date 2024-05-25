# 10 Tips For Building SSR/HDA applications - Apuntes

## Tip 1: Maximize Your Server-Side Strengths

A big advantage of the hypermedia-driven approach is that it makes the server-side environment far more important when building your web application. Rather than simply producing JSON, your back end is an integral component in the user experience of your web application.


## Tip 2: Factor Your Application On The Server
> hypermedia-driven applications are typically organized via template inclusion, where the server-side templates are broken up according to the HTML-rendering needs of the application

La ventaja de esto es que el template contiene el HTML suficiente para mostrar la vista o parte de ella.


## Tip 3: Specialize Your API End Points
> the hypermedia API you produce for your hypermedia-driven application should feature end-points specialized for your particular application’s UI needs.
> 
> Your end-points should be optimized to support your particular applications UI/UX needs, not for a general-purpose data-access model for your domain model.

Tienen que ser endpoints de aplicación. Que devuelvan todo el HTML para una misma vista.


## Tip 4: Aggressively Refactor Your API End Points
> A great strength of the hypermedia approach is that you can completely rework your API to adapt to new needs over time without needing to version the API or even document it.

De esta forma es más sencillo actualizar los templates porque son casi que uno-a-uno con su vista. No causan dependencias.


## **Tip 5: Take Advantage of Direct Access To The Data Store**
> Because your hypermedia API can be structured around your UI needs, you can tune each endpoint to issue as few data store requests as possible.

Puedes hacer que cada endpoint devuelva toda la información necesaria de una sola consulta. No hay que mezclar endpoints individuales para generar un todo.


## Tip 6: Avoid Modals

Consider using alternatives such as [inline editing](https://htmx.org/examples/click-to-edit/), rather than modals.


## **Tip 7: Accept “Good Enough” UX**
> Accepting a slightly less efficient and interactive solution to a particular UX can save you a tremendous amount of complexity when building a web application.


## Tip 8: When Necessary, Create “Islands of Interactivity”

Como vimos en  [+Hypermedia-Friendly Scripting - Apuntes](https://paper.dropbox.com/doc/Hypermedia-Friendly-Scripting-Apuntes-d5RAHukg8jYh8DEW5jlcl) 


## **Tip 9: Don’t Be Afraid To Script!**

Cuando se necesiten scripts, se escriben scripts. No hay problema mientras estos no cambien el estado en el servidor. Eso debe hacerlo Hypermedia.


## Tip 10: Be Pragmatic
> do not be dogmatic about using hypermedia. At the end of the day, it is just another technology with its own strengths & weaknesses.

