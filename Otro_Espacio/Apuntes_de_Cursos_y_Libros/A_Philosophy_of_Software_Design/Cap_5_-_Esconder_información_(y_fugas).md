# Cap.5 - Esconder información (y fugas)

## Information Hiding
> The basic idea is that each module should encapsulate a few pieces of knowledge, which represent design decisions. The knowledge is embedded in the module’s implementation but does not appear in its interface, so it is not visible to other modules.

La información que se esconde en el módulo corresponde a detalles de cómo se implementa un mecanismo. Ejemplos:

- Cómo almacenar información en un árbol binario, y cómo acceder a esta.
- Cómo implementar el protocolo de red TCP

Ocultar información reduce la complejidad de dos formas:

Primero, al simplificar la interfaz de un módulo.


> The interface reflects a simpler, more abstract view of the module’s functionality and hides the details;

de esa forma se reduce al carga cognitiva de los desarrolladores que usen el módulo.

Segundo, al hacer más fácil evolucionar un sistema.


> If a piece of information is hidden, there are no dependencies on that information outside the module containing the information, so a design change related to that information will affect only the one module.

**Nota: esconder variables y métodos en una clase mediante la palabra clave** `**private**` **no es lo mismo a** ***esconder información*****.**

**La mejor forma de esconder información es cuando esta está totalmente oculta en el módulo, de tal forma que es irrelevante e invisible a los usuarios del módulo**.

¿Cómo se logra esto?


## Information Leakage

Ocurre cuando una decisión de diseño se refleja en varios módulos. Esto crea una dependencia entre los módulos y cualquier cambio deberá hacerse en varias partes.


> If a piece of information is reflected in the interface for a module, then by definition it has been leaked;


> One of the best skills you can learn as a software designer is a high level of sensitivity to information leakage.

Una forma de solucionar una fuga de información presente en varias clases es creando una clase nueva que tenga toda la información y la encapsule. Solo sirve si la abstracción es más sencilla.


## Temporal Decomposition

Ocurre cuando la estructura de un sistema corresponde al orden temporal en el que las operaciones suceden.


> With temporal decomposition, this application might be broken into three classes: one to read the file, another to perform the modifications, and a third to write out the new version.


> This is an example of a temporal decomposition (“first we read the request, then we parse it”).


> When designing modules, focus on the knowledge that’s needed to perform each task, not the order in which tasks occur.


> information hiding can often be improved by making a class slightly larger.


## Information hiding within a class

Intenta diseñar los métodos privados de una clase de tal forma que cada método encapsula algo de información y la esconde del resto de la clase.

Intenta minimizar el número de lugares donde cada variable de instancia se usa.

Si logras eliminar el número de lugares donde una variable de instancia se usa, eliminarás dependencias dentro de la clase y eso reduce su complejidad.


> Information hiding only makes sense when the information being hidden is not needed outside its module.


## Conclusion
> If a module hides a lot of information, that tends to increase the amount of functionality provided by the module while also reducing its interface.


> When decomposing a system into modules, try not to be influenced by the order in which operations will occur at runtime;

En vez del orden mejor:

> Instead, think about the different pieces of knowledge that are needed to carry out the tasks of your application, and design each module to encapsulate one or a few of those pieces of knowledge.

