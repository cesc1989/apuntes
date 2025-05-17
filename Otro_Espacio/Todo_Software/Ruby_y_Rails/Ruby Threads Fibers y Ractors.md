# Ruby Threads, Fibers y Ractors

¿Qué carajo son? ¿Para qué sirven? ¿En qué se diferencian?

## Un poco de contexto

Del subreddit [r/ruby](https://www.reddit.com/r/ruby/comments/eh69or/eli5_what_are_ruby_thread_process_fiber/):

> So all "Process", "Thread" and "Fiber" are concepts describing a thread of execution, a series of instructions your computer will execute one after the other.

Otro comentario dice que:

> Processes = A separate program execution. Heavy, relatively resource intensive, CPU needs to context switch
>
> Thread = Parallel execution inside a Process. More lightweight than a Process but shares the same address space as the spawning (parent) Process.
>
> Fiber = Lightweight parallel execution inside a Thread.

# Threads

Definición de [Wikipedia](https://es.wikipedia.org/wiki/Hilo_(inform%C3%A1tica)):

> En sistemas operativos, un hilo (...) es una secuencia de tareas encadenadas (...) que puede ser ejecutada por un sistema operativo. 

Agrega que:

> Un hilo es simplemente una tarea que puede ser ejecutada al mismo tiempo que otra tarea.

## ¿Para qué los _threads_?

El artículo de Wikipedia y el [libro de Jesse Storimer](https://workingwithruby.com/wwrt/intro/) dicen que una ventaja de los hilos es que comparten memoria por lo tanto levantar más hilos es más barato que levantar más procesos. Los procesos no comparten memoria.

Al final, los hilos ofrecen más unidades de trabajo que los procesos a menor carga y consumo de recursos del sistema. Sin embargo, no son gratis. Para poder hacer esto se necesita que el programa que levanta muchos hilos sea _thread safe_.

## ¿Qué es Thread Safe

Según Jesse Storimer en _Working with Ruby Threads_:

> If your code is 'thread-safe,' that means that ==it can run in a multi-threaded context and your underlying data will be safe==. By data, I'm not talking about what's in your database; I'm talking about what values your program has stored in memory. Your data is what's at stake.

## Threads en Ruby

Storimer dice que, por defecto, todo código Ruby corre sobre un hilo. Aún cuando no se crean hilos nuevos el código se ejecuta sobre el _main thread_.

```ruby
$ irb
irb(main):001> Thread.main
=> #<Thread:0x000000010153b108 run>
```

Cuando se quieren usar hilos en programas en Ruby hay unas preguntas cruciales que hacerse:

- ¿Dónde y cuándo debe levantarse un nuevo hilo?
- ¿Cuántos hilos hay que levantar?
- ¿Qué pasa con _thread safety_?

Los hilos son poderosos por sus beneficios de concurrencia a bajo coste pero como dijo el tío Ben: _todo gran poder conlleva una gran responsabilidad_.

### Shared Address Space o Referencias en Memoria

El libro de Storimer dice que lo más importante cuando se trabaja con hilos el _shared address space_. Multiples hilos tendrán acceso a las mismas referencias en memoria (variables) así como al AST (_Abstract Syntax Tree_) que viene siendo el código fuente ya compilado.

Otra trampa el programar con hilos está en que estamos acostumbrados a pensar y escribir nuestro código pensando en un único hilo:

- Un método ejecuta una función
- Si la función devuelve true hace aquello
- Y luego de eso se manda un mensaje
- etc

Esto lo podemos llamar _hilo de ejecución_. Un posible camino que puede ser atravesado por el código. La cosa se complica si introducimos varios hilos que pueden plantear diferentes hilos de ejecución al mismo tiempo.

## Concurrencia no es igual a Paralelismo

Jesse dedica un [capítulo a explicar esta diferencia](https://workingwithruby.com/wwrt/concurrent_vs_parallel/). Lo hace porque es crucial en el mundo Ruby y los hilos.

Los hilos permiten ejecución concurrente pero no en paralelo. Quiere decir que mediante los hilos se pueden ir haciendo varias cosas pero no al mismo tiempo.

Aclara que:

> multi-threaded code should be referred to as concurrent

Finalmente, esto es importante porque en Ruby, al menos en el interprete de Matz, el código se puede ejecutar de manera concurrente pero no en paralelo. Esto por la existencia del Global Interpreter Lock, el famoso GIL.

El GIL es infame en Ruby porque impide la ejecución en paralelo de los hilos. El libro explica que:

- Solo hay un GIL por proceso de MRI
- Los hilos de ese proceso comparten el mismo GIL
- Si un hilo quiere ejecutar código ruby, necesita pedir un _lock_ al GIL
- Los demás hilos esperan mientras se libera el _lock_

**Todo esto significa que el GIL previene la ejecución de código en paralelo**.

## Threads en Rails

Puma y Sidekiq son librerías que hacen multi-hilo.

La clave para escribir aplicaciones multi-hilo que sean thread-safe en Rails es no compartir estados globales. Cada petición debe tener sus propias variables y no leer de estados globales.

> ==make sure that each controller action creates the objects that it needs to work with==, rather than sharing them between requests via a global reference.

Jesse desaconseja tener algo como `User.current`.

En Rails tenemos [CurrentAttributes](https://api.rubyonrails.org/classes/ActiveSupport/CurrentAttributes.html) que bien podría ir en contra de esto pero la documentación dice que:

> Abstract super class that provides a thread-isolated attributes singleton, which resets automatically before and after each request.


# Fibers

Empecemos con un poco de contexto. Esta [respuesta](https://stackoverflow.com/a/9194052/1407371) en Stack Overflow de hace más de 10 años.

> Fibers are something you will probably never use directly in application-level code. They are a flow-control primitive which you can use to build other abstractions, which you then use in higher-level code.

Vamos a ver si eso ha cambiado.

## Ruby Fibers 101

Artículo de [Saeloun](https://blog.saeloun.com/2022/03/01/ruby-fibers-101/)

Los Fibers existen desde Ruby 1.9. Es desde Ruby 3 que vienen a tener más relevancia por la introducción de [Fiber::SchedulerInterface](https://rubyapi.org/3.0/o/fiber/schedulerinterface). Esta nueva clase permite cambio de contexto de manera rápida y más eficiente que los Threads.

Los Fibers son workers ligeros similares los Threads pero con la diferencia de ser más eficiente en el uso de memoria y permitirle al programador controlar cuando el código debe pausar y reanudar.

A nivel de aplicación los Fibers permiten esperas en I/O no bloqueante. Esto significa que cuando una Fiber está accediendo a un servicio I/O puede pasarle el control a otro Fiber para que complete su trabajo. Una vez el segundo Fiber termine, el primero puede retomar por donde iba.

### Contexto Tradicional: OS Thread

En sistemas normales, el bloqueo de I/O se da al asignar un Thread de Sistema Operativo (OS Thread) a cada petición. Esto presenta un desafío en que los OS Threads son más caros de crear y necesitan de cambio de contexto (cambio de datos, memoria) por parte del Sistema Operativo cada que un nuevo hilo deba ejecutarse.

Todo lo anterior es bastante pesado para el sistema.

Los Fibers palian tal sobrecarga ya que fueron creados para ser ligeros y consumir poca memoria. Esto lo logran al operar en un mismo Thread.

Como regla al menos hay un Fiber por Thread. [En Ruby] siempre hay un Fiber activo.

> [!Tip]
> Aún cuando no se están usando Fibers ni haciendo trabajo relacionado, el código se ejecuta en el contexto de un main Fiber en el Thread actual. Todo creado automáticamente por Ruby por cada Thread.

> [!Important]
> Al usar Fibers el programador indica cuando empieza y cuando se detiene su ejecución.

### Fibers vs Threads

Ruby MRI usa un _fair scheduler_ el cual le asigna a cada Thread una cantidad igual de tiempo de ejecución antes de que pause y le pase el control al siguiente thread.

Si tenemos dos Threads y uno ya está esperando I/O, cuando el scheduler le ceda el control no hará nada al estar esperando. Esto significaría perdida de procesamiento.

Al contrario, los Fibers al ser pausados y reanudados según decida el programador, son más flexibles con el uso de los recursos del CPU. Eso sí, pueden hacer más complejos los programas que hagan uso de esta herramienta.

### No son la solución al paralelismo

El problema principal de Ruby es el GIL. Threads y Fibers permiten concurrencia pero no paralelismo. Los Fibers son un poco mejor y tienen otros usos que los Threads pero ambos ayudan a la misma tarea.

La verdadera solución al paralelismo son los [Ractors](https://github.com/ruby/ruby/blob/master/doc/ractor.md).

# Ractors

> Ractor is an Actor-Model like concurrent abstraction designed to provide a parallel execution without thread-safety concerns. Ractors allow threads in different ractors to compute at the same time. Each ractor has at least one thread, which may contain multiple fibers. Inside a ractor, only a single thread is allowed to execute at a given time.
>
> Saeloun