# Cap.12 - Why Write Comments?
La documentación juega un rol importante al crear abstracciones. Sin comentarios, no puedes ocultar la complejidad.

El proceso de escribir comentarios, hecho correctamente, mejorará el diseño del sistema. Un buen diseño de sistema pierde valor si está pobremente documentado.

![](https://paper-attachments.dropboxusercontent.com/s_DF275FF5F9B3926EBEAEAD45B59944822357F4AF7C527E02C2C271C2A4AE88B1_1685377197227_imagen.png)



# Good code is self-documenting.
> This is a delicious myth, like a rumor that ice cream is good for your health: we’d really like to believe it! Unfortunately, it’s simply not true. To be sure, there are things you can do when writing code to reduce the need for comments, such as choosing good variable names.

Hay cosas que no pueden ser descritas por el código, como lo puede ser los motivos por los cuales una decisión de diseño particular existe, o las condiciones en las cuales tiene sentido usar un método en específico.


> if you write code with the expectation that users will read method implementations, you will try to make each method as short as possible, so that it’s easy to read. If the method does anything nontrivial, you will break it up into several smaller methods.
> 
> This will result in a large number of shallow methods. Furthermore, it doesn’t really make the code easier to read: in order to understand the behavior of the top-level method, readers will probably need to understand the behaviors of the nested methods.

Métodos llanos (*shallow methods*) en exceso es malo.


> comments are fundamental to abstractions. (…) the goal of abstractions is to hide complexity: an abstraction is a simplified view of an entity, which preserves essential information but omits details that can safely be ignored.
> 
> If users must read the code of a method in order to use it, then there is no abstraction: all of the complexity of the method is exposed.


# I don’t have time to write comments
> If you want a clean software structure, which will allow you to work efficiently over the long-term, then you must take some extra time up front in order to create that structure. 
> 
> Good comments make a huge difference in the maintainability of software, so the effort spent on them will pay for itself quickly.

Escribir comentarios no tomará más del 10% del tiempo que se gasta escribiendo código.

# Comments get out of date and become misleading

Sí, los comentarios quedan desactualizados pero esto no debería ser un problema mayor.

Los grandes cambios a la documentación son solo necesarios si hubo grandes cambio en el código. Además, los cambios de código tomarán mucho más tiempo que los cambios en documentación.

Mantén los comentarios cerca del código pertinente y será más fácil tenerlos a la vista para actualizarlos.

# All the comments I have seen are worthless

Es cierto pero podemos implementar un sistema para que los comentarios tengan valor.


# Benefits of well-written comments
> The overall idea behind comments is to capture information that was in the mind of the designer but couldn’t be represented in the code.

Detalles que pueden servir al próximo desarrollador a entender porqué el código está así y no el qué hace.


> Documentation can reduce cognitive load by providing developers with the information they need to make changes and by making it easy for developers to ignore information that is irrelevant.


> Documentation can also reduce the unknown unknowns by clarifying the structure of the system, so that it is clear what information and code is relevant for any given change.


# A different opinion: comments are failures
> comments do not represent failures. The information they provide is quite different from that provided by code, and this information can’t be represented in code today.


> One of the purposes of comments is to make it it unnecessary to read the code: for example, instead of reading the entire body of a method, a developer can read a short interface comment to get all the information they need in order to invoke the method.


> Well-written comments are not failures. They increase the value of code and serve a fundamental role in defining abstractions and managing system complexity.


