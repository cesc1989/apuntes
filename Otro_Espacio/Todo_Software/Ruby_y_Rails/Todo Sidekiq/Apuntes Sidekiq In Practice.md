# Apuntes de Sidekiq in Practice de Nate Berkopec

Etiquetas: #sidekiq 

## Capítulo 1: How Sidekiq Works, and Why

Sidekiq es una cola FIFO. Primero en entrar, primero en salir:
> “The threads in a Sidekiq server pull jobs from one or more queues and perform the work. They pull they oldest job from the queues they are assigned to process; Sidekiq is “first in, first out” (FIFO).”

Sidekiq opera enviando comandos a Redis para guardar las colas. La cantidad de comandos, según como se encole, importa.
> “While you don’t have to understand these commands or what they do, exactly, you should generally pay attention to the number of Redis commands used when working with Sidekiq.”

El encolado que implica dos comandos de Redis tarda más que la opción de un solo comando:
> “Sidekiq’s Redis commands are generally executed serially, so we wait for a reply from the database before sending the next command. That means that 2 commands generally take 2 times as long to execute. It also means that we’re imposing 2 times as much load on the Redis database”

Clave las opciones de encolado que implican un solo comando de Reds:
> “Because Redis is single-threaded and so can only run one command at a time, ==fewer Redis commands means less Redis load, which means more jobs per second.==”

## Capítulo 2: Understanding Queueing Systems

Este capítulo explica terminología alrededor de los sistemas de colas. Explica algunos términos como:

**Unit of Work (unidad de trabajo)**: En Sidekiq, esto es un _job request_.

**Server**: En teoría de colas, un servidor es "una unidad de capacidad de procesamiento". Si tengo 1 "servidor", este podrá procesar una unidad de trabajo a la vez.

**Service Time**: esto es cuánto toma procesar una unidad de trabajo. O sea, cuanto tiempo tarda en correr un job en Sidekiq.

**Wait Time**: Es cuánto tiempo las unidades de trabajo se quedan esperando en la cola.

**Total Time**: Esto es _service time_ + _wait time_. Es el tiempo total que tarda un job en ser encolado hasta que termina.

### Queue Length Exponentially Increases as Utilization Increases

La cola incrementa de manera exponencial a medida que la utilización aumenta:
> “we can divide queueing systems into two regimes: low utilization and high utilization. ==When utilization of the system is between 0 and 70%, queue wait times increase quite slowly as utilization increases==. But, ***when utilization increases beyond 70%, the exponential effects take over and queue latency increases quickly.***”

### Wait Time is Proportional to 1/Servers

Que haya alta ocupación es positivo ya que se están usando los recursos.
> “High utilization is generally a good thing. It means we’re not paying for any excess capacity. We generally want utilization to be as high as possible, while at the same time making sure queue latency doesn’t exceed our *requirements*”

Detalle importante. Una cola que tiene varios servidores/procesos halando de ella es mejor en términos de utilización de recursos.
> “Queues which have more servers pulling from them are generally better for utilization and cost efficiency than queues which have just a few servers pulling from them.
> (...) more processes pulling from one queue - means lower average wait times, thanks to the 1/servers rule of thumb”

### Requerimientos

Es los requerimientos mencionados antes en cursiva. Se refiere a los requerimientos necesarios para diseñar un sistema. Se pueden dividir en tres categorías.

**Buena experiencia de cliente**: explica que la latencia del job depende del flujo/proceso al cual el cliente se enfrenta. Si es reiniciar una contraseña se espera que sea rápido. Si es actualizar algunos datos, podría tomar horas.

Clave aquí es:
> [!Important]
> Todos los trabajos en segundo plano tienen un requerimiento en la latencia. Si no tuvieran, no habría razón para ejecutarlos.

**Un sistema eficiente en recursos y escalable**: se refiere a que el sistema pueda designar más recursos para procesar la cola siendo eficiente con el dinero y además que sea automático.

**Un sistema estable cuando la carga incrementa rápidamente**: Esto es lo que pasa en Alpha en Luna.

> “Rapid growth of a queue can effectively cause a “brownout”, where the system is not technically down (it’s still online and processing jobs), but the queue has become so deep that by the time jobs execute, they may be completely irrelevant!”

La cola default crece tan rápido que los jobs tardan mucho en ser procesados y cuando lo son ya no importa.

### USE method

Es una forma de establecer como lograr el sistema Sidekiq ideal. Son tres aristas:

- Utilization
- Saturation
- Errors

#### Utilization

> “In a queueing system, the most important resources are not memory or CPU, but the servers which can do work. Remember, I mean “servers” in the queueing theory sense here, not the computing sense”

> “So what percent utilization is “good”? Well, generally, higher is better, because any part of the system that we’re not utilizing is increasing costs without any benefit.”

> “we can’t just run at 99% utilization all the time, because it will send our next metric, saturation, in the wrong direction.”

#### Saturation

> “Saturation is what occurs when our system is 100% utilized at any given moment: that is, saturation is the growth of the queue.”

> “Saturation is one of the most important metrics in our system because it’s the primary component of latency experienced by the customer."

> "Most jobs take less than one second to execute, but can spend minutes in a queue." 

> We can control saturation by decreasing utilization - that is, either decreasing incoming work or by adding additional resources to process it.”

#### Errors

> “All errors are wasted capacity: wasted work that could have been spent doing something productive. Keeping errors low means we keep our server capacity available for useful work.”

Esto se puede identificar con métrica de errores:

- Tamaño de la cola retry
- Tamaño de la cola muertos
- Cantidad de errores de conexión a Redis

La clave es que estas sean lo menor posible para que la capacidad se gaste en trabajo útil.

### El Sidekiq Ideal

1. El tiempo total de cada job es menos del indicado por el requerimiento.
2. Utilización es tan alta como sea posible y aún así cumpliendo los requerimientos.
3. Los errores están al mínimo para que la capacidad se gaste en trabajo útil.
4. El sistema puede responder rápidamente a los cambios drásticos de la cola.


## Capítulo 3: Setting Concurrency

