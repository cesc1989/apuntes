# Cap.9 - Better Together or Better Apart?

> One of the most fundamental questions in software design is this: given two pieces of functionality, should they be implemented together in the same place, or should their implementations be separated? This question applies at all levels in a system, such as functions, methods, classes, and services.

Subdividir puede crear otro tipo de problemas

> the act of subdividing creates additional complexity that was not present before subdivision:

Complejidad por el número de componentes

> the more components, the harder to keep track of them all and the harder to find a desired component within the large collection.
> 
> Every new interface adds complexity.

Al subdividir se obteien más código para manejar componentes del sistema

Al subdividir se crea separación: el componente subdividido estará muy lejor de lo que estaba antes.

> If the components are truly independent, then separation is good: it allows the developer to focus on a single component at a time, without being distracted by the other components.


> if there are dependencies between the components, then separation is bad: developers will end up flipping back and forth between the components.

Al subdividir se puede terminar duplicando código porque se necesita en todas las subdivisiones.


> Bringing pieces of code together is most beneficial if they are closely related. If the pieces are unrelated, they are probably better off apart.

**Indicaciones de que dos piezas de código están relacionadas**

- Comparten información: ejemplo, el código depende de la sintaxis de un tipo de documento
- Se usan en conjunto: quien use una pieza probablemente usará la otra. Solo mantener en conjunto si la dependencia es bidireccional.
- Se solapan conceptualmente: hay una categoría superior que las incluye
- Es difícil entender una pieza sin ver la otra
## Bring together to eliminate duplication
> This approach is most effective if the repeated code snippet is long and the replacement method has a simple signature.


> If the snippet is only one or two lines long, there may not be much benefit in replacing it with a method call.
## Separate general-purpose and special-purpose code
> If a module contains a mechanism that can be used for several different purposes, then it should provide just that one general-purpose mechanism. It should not include code that specializes the mechanism for a particular use, nor should it contain other general-purpose mechanisms.

**Red Flags**
Repetition

![](https://paper-attachments.dropboxusercontent.com/s_A7CB480A863B8877C5585E37B5062F568FC3042211F468EF0F155D4567A85D10_1679002075348_imagen.png)


Special-General Mixture

![](https://paper-attachments.dropboxusercontent.com/s_A7CB480A863B8877C5585E37B5062F568FC3042211F468EF0F155D4567A85D10_1679002102069_imagen.png)

## Splitting and joining methods
> The issue of when to subdivide applies not just to classes, but also to methods: are there times when it is better to divide an existing method into multiple smaller methods?


> length by itself is rarely a good reason for splitting up a method. 
> 
> Splitting up a method introduces additional interfaces, which add to complexity.

No dividas métodos a menos que hagan al sistema más simple

> If the blocks are relatively independent, then the method can be read and understood one block at a time; there’s not much benefit in moving each of the blocks into a separate method.
> 
> If the blocks have complex interactions, it’s even more important to keep them together so readers can see all of the code at once;
> 
> Methods containing hundreds of lines of code are fine if they have a simple signature and are easy to read.

**Cómo y cómo no dividir métodos**

![](https://paper-attachments.dropboxusercontent.com/s_A7CB480A863B8877C5585E37B5062F568FC3042211F468EF0F155D4567A85D10_1679002473913_imagen.png)


Todo método debe hacer una cosa y hacerla enteramente. El método debe tener una interfaz sencilla de tal manera que los usuarios no necesitan tanta informació en la cabeza para poder usarla correctamente.

**Figure 9.3 (b)**

> This form of subdivision makes sense if there is a subtask that is cleanly separable from the rest of the original method, which means (a) someone reading the child method doesn’t need to know anything about the parent method and (b) someone reading the parent method doesn’t need to understand the implementation of the child method.

**Figure 9.3 (c)**

> If you make a split like this, the interface for each of the resulting methods should be simpler than the interface of the original method. Ideally, most callers should only need to invoke one of the two new methods; if callers must invoke both of the new methods, then that adds complexity, which makes it less likely that the split is a good idea.
> 
> don’t make sense very often, because they result in callers having to deal with multiple methods instead of one. When you split this way, you run the risk of ending up with several shallow methods,

Y a veces es mejor juntar métodos

> joining methods might replace two shallow methods with one deeper method; it might eliminate duplication of code; it might eliminate dependencies between the original methods, or intermediate data structures; it might result in better encapsulation, so that knowledge that was previously present in multiple places is now isolated in a single place;


## A different opinion: Clean Code
> A more important issue is: does breaking up a function reduce the overall complexity of the system?

Muchas funciones pequeñas no necesariamente es mejor:

> More functions means more interfaces to document and learn. If functions are made too small, they lose their independence, resulting in conjoined functions that must be read and understood together.

Finalmente:

> Depth is more important than length: first make functions deep, then try to make them short enough to be easily read. Don’t sacrifice depth for length.

