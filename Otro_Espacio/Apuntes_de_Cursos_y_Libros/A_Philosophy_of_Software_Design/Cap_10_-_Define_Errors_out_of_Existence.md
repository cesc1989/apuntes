# Cap.10 - Define Errors out of Existence

> Code that deals with special conditions is inherently harder to write than code that deals with normal cases, and developers often define exceptions without considering how they will be handled.


## Why exceptions add complexity
> However, exceptions can occur even without using a formal exception reporting mechanism, such as when a method returns a special value indicating that it didn’t complete its normal behavior. All of these forms of exceptions contribute to complexity.


> Exception handling code is inherently more difficult to write than normal-case code. An exception disrupts the normal flow of the code; it usually means that something didn’t work as expected and an operation cannot be completed as planned.


> Language support for exceptions tends to be verbose and clunky, which makes exception handling code hard to read.


## Too many exceptions
> Programmers exacerbate the problems related to exception handling by defining unnecessary exceptions.


> This leads to an over-defensive style where anything that looks even a bit suspicious is rejected with an exception,

Cuándo no se tiene clara la idea de cómo resolver algo, se usa una excepción y se le pasa el problema al siguiente que use la clase o módulo (está mal)?

> It’s tempting to use exceptions to avoid dealing with difficult situations: rather than figuring out a clean way to handle it, just throw an exception and punt the problem to the caller. 
> 
> Generating an exception in a situation like this just passes the problem to someone else and adds to the system’s complexity.



> The best way to reduce the complexity damage caused by exception handling is to reduce the number of places where exceptions have to be handled.


## Define errors out of existence
> The best way to eliminate exception handling complexity is to define your APIs so that there are no exceptions to handle: define errors out of existence.

Si el código gestiona los casos extremos no hay necesidad de controlar excepciones

> The Java `substring` method would be easier to use if it performed this adjustment automatically, so that it implemented the following API: “returns the characters of the string (if any) with index greater than or equal to `beginIndex` and less than `endIndex`.” 
> 
> This is a simple and natural API, and it defines the `IndexOutOfBoundsException` exception out of existence. The method’s behavior is now well-defined even if one or both of the indexes are negative, or if `beginIndex` is greater than `endIndex`.

Mejor buscar programar para eliminar errores que para capturarlos

> The error-ful approach may catch some bugs, but it also increases complexity, which results in other bugs.
> 
> In contrast, defining errors out of existence simplifies APIs and it reduces the amount of code that must be written. Overall, the best way to reduce bugs is to make software simpler.


## Mask exceptions
> With this approach, an exceptional condition is detected and handled at a low level in the system, so that higher levels of software need not be aware of the condition.

Ejemplos:

> TCP masks packet loss by resending lost packets within its implementation, so all data eventually gets through and clients are unaware of the dropped packets.


> If an NFS file server crashes or fails to respond for any reason, clients reissue their requests to the server over and over again until the problem is eventually resolved. The low-level file system code on the client does not report any exceptions to the invoking application.

¿por qué es una buena alternativa?

> the best alternative is for NFS to mask the errors and hang applications. With this approach, applications don’t need any code to deal with server problems, and they can resume seamlessly once the server comes back to life.

Pero no es para todos los casos

> Exception masking doesn’t work in all situations, but it is a powerful tool in the situations where it works. It results in deeper classes, since it reduces the class’s interface (fewer exceptions for users to be aware of) and adds functionality in the form of the code that masks the exception.
> 
> Exception masking is an example of pulling complexity downward.


## Exception aggregation
> Instead of catching the exceptions in the individual service methods, let them propagate up to the top-level dispatch method


> The top-level exception handler encapsulates knowledge about how to generate error responses, but it knows nothing about specific errors; it just uses the error message provided in the exception.


> Exception aggregation works best if an exception propagates several levels up the stack before it is handled; this allows more exceptions from more methods to be handled in the same place.
> 
> This is the opposite of exception masking: masking usually works best if an exception is handled in a low-level method.

Cómo se suele hacer en controladores en rails

> One way of thinking about exception aggregation is that it replaces several special-purpose mechanisms, each tailored for a particular situation, with a single general-purpose mechanism that can handle multiple situations. This provides another illustration of the benefits of general-purpose mechanisms.


## Just crash?
> The fourth technique for reducing complexity related to exception handling is to crash the application. In most applications there will be certain errors that are not worth trying to handle.



