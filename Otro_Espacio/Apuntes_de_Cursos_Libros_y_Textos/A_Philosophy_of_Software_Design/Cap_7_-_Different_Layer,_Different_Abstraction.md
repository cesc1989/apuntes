# Cap.7 - Different Layer, Different Abstraction

> Software systems are composed in layers, where higher layers use the facilities provided by lower layers. In a well-designed system, each layer provides a different abstraction from the layers above and below it


## Pass-through methods

Algo que me ha pasado en proyectos como Lupita, Bucket e incluso Luna 😁 

> When adjacent layers have similar abstractions, the problem often manifests itself in the form of pass-through methods. A pass-through method is one that does little except invoke another method,


> Pass-through methods make classes shallower: they increase the interface complexity of the class, which adds complexity, but they don’t increase the total functionality of the system.

¿cómo se arregla?

> The solution is to refactor the classes so that each class has a distinct and coherent set of responsibilities.


## When is interface duplication OK?
> The important thing is that each new method should contribute significant functionality. Pass-through methods are bad because they contribute no new functionality.

La duplicación está bien en métodos tipos *dispatcher*.

> One example where it’s useful for a method to call another method with the same signature is a dispatcher.


## Decorators
> A decorator object takes an existing object and extends its functionality; it provides an API similar or identical to the underlying object, and its methods invoke the methods of the underlying object.


> The motivation for decorators is to separate special-purpose extensions of a class from a more generic core.


## Pass-through variables
> Another form of API duplication across layers is a pass-through variable, which is a variable that is passed down through a long chain of methods.


> Pass-through variables add complexity because they force all of the intermediate methods to be aware of their existence, even though the methods have no use for the variables.

¿cómo solucionar?

> The solution I use most often is to introduce a context object. A context stores all of the application’s global state (anything that would otherwise be a pass-through variable or global variable).
![](https://paper-attachments.dropboxusercontent.com/s_A7CB480A863B8877C5585E37B5062F568FC3042211F468EF0F155D4567A85D10_1678917368794_imagen.png)

> In (d), cert is stored in a context object along with other system-wide information, such as a timeout value and performance counters; a reference to the context is stored in all objects whose methods need access to it.

Tiene su pero

> Unfortunately, the context will probably be needed in many places, so it can potentially become a pass-through variable. To reduce the number of methods that must be aware of it, a reference to the context can be saved in most of the system’s major objects.

El objeto de contexto unifica toda la información global

> The context object unifies the handling of all system-global information and eliminates the need for pass-through variables. If a new variable needs to be added, it can be added to the context object;

No son la solución ideal pero es la “mejorcita”

> Contexts are far from an ideal solution. The variables stored in a context have most of the disadvantages of global variables;
> 
> Contexts may also create thread-safety issues; the best way to avoid problems is for variables in a context to be immutable.


## Conclusion

Como dicen en "Getting Real" : código que se agrega, código que hay que mantener y cuidar:

> Each piece of design infrastructure added to a system, such as an interface, argument, function, class, or definition, adds complexity, since developers must learn about this element.

La complejidad que cada cosa agrega debe balancearse con la utilidad que provee:

> In order for an element to provide a net gain against complexity, it must eliminate some complexity that would be present in the absence of the design element.

