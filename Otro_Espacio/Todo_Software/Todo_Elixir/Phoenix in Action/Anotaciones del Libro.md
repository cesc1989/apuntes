# Anotaciones del Libro

## Sobre Phoenix Channels

>  Phoenix has the concept of channels built in. Channels allow for “soft-realtime” features to be easily added to applications. (...) they allow you to have chat applications, instantly push updates to thousands of simultaneous users, and update pages without a full-page refresh.


## Supervision trees

> Elixir applications (including Phoenix apps) have supervisors and workers. Supervisors monitor the workers and ensure that if they go down for one reason or another, they’re started right back up. This provides an amazing backbone of fault-tolerance without much work on your part.


## OTP

> OTP is a set of functions, modules, and standards that make using things such as supervisors and workers—and the fault-tolerance that goes along with them—possible. OTP includes concepts such as servers, concurrency, distributed computation, load balancing, and worker pools, to name just a few.

## Functions

> There are two main kind of functions in Elixir: named functions and anonymous functions. Most of the time you'll write named functions.
>
> Named functions in Elixir are always within the context or scope of a module. A named function can’t exist outside of a module.

## Recordar cómo acceder a funciones

> To recall the structure of a function call, you can remember MFA—module-function-arguments.

Gráfico:

![[elixi.mfa.png]]

## Sobre Tuples

> Tuples are not meant to be used as a “collection” type (...): they’re mostly meant to be used as a fixed-size container for multiple elements.

## Las vistas en Phoenix no son como las vistas en Rails

> The view is responsible for rendering templates and for setting up helper functions that you can use in those templates. ==The functions defined here are similar to decorators or presenters in other frameworks==.

