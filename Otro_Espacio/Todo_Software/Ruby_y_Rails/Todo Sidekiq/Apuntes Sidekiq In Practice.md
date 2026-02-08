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

> [!Note]
> Este capítulo tiene muy buenos detalles.

---

> “Using multiple threads in one process increases job throughput by a factor of 50% to 100% over single threaded processes for most workloads.”

> “Higher concurrency means higher throughput, but can also lead to excessive latency and memory use.”

> “Set concurrency too low, and you’re leaving additional throughput on the table”

> “**fewer threads is often better.**”

> “With Sidekiq 5.2.0, the default concurrency setting was changed from 25 to 10. Why? The community (and, then Mike) learned, in the meantime, about two major costs: memory and GVL contention.”

> “only one thread can run Ruby code at a time. That is, _in order to run Ruby code, the thread must hold the Global VM Lock._”

> “==When there is more than one thread in a process, multiple threads can be ready to run Ruby code, but only one can run it at a time==. **This creates queueing for the Global VM Lock, as threads wait for the GVL to free up.** As we covered in the previous chapter, queueing increases latency.”

> “concurrency also interacts with database pools: **if concurrency is significantly higher than the size of one’s database pool, we can cause additional latency there as well** (...) If all of the connections are already checked out, queueing occurs, increasing latency. (...) ==Setting concurrency higher than your database pool setting can slow down your Sidekiq by 100x.==”

### Costos en memoria

> “A single Sidekiq process can slowly grow from 256MB of memory usage to 1GB in less than 24 hours. (...) this is actually memory fragmentation.”

> “The more fragmented the memory heap, the more memory it will consume. This is because allocating an object requires the right size “hole” in the heap, and in a fragmented heap, those holes are too small, requiring the growth of the heap in order to satisfy the allocation.”

> “==What we’ve learned since Sidekiq became popular is that memory fragmentation becomes much worse in a direct relationship with the number of Ruby threads.==”

> “what usually limits Sidekiq throughput in the real world is contention for the Global VM Lock”

### Disputa por el Global VM Lock

Por cada thread adicional que se agregue al proceso las mejoras se van reduciendo:
> “What we’re sure of is that there are greatly diminishing returns - in fact, we’re mathematically sure of it. The second thread adds more throughput than the third, which adds more throughput than the fourth, and so on.”

> “But our threads perform other work which is not running Ruby code and does not require the Ruby Virtual Machine. The biggest task in this category is I/O: sending and receiving data, particularly across the network. This is because I/O in the CRuby runtime is implemented in C code, not in Ruby. We don’t need the Ruby VM or its lock to run C.”

> “10 threads is a great default for most apps using Sidekiq, but 5 to 128 is a reasonable range. ”

### Proceso para configurar `concurrency`

1. “For each Sidekiq “worker type” (that is, a particular configuration of Sidekiq which pulls from a certain set of queues), determine the average % of time spent in I/O. Use your APM (New Relic, etc) for this. Be sure to exclude jobs from your calculation that this worker type does not process.”

2. “Use the table at the end of this chapter to find a “starting point” based on I/O workload”

3. “Deploy with this setting. If memory usage exceeds acceptable limits, adjust concurrency downward until it is acceptable, or change your server type to one with more memory available.”

4. “If service time (latency observed in your APM) increases significantly as a result of the concurrency change, decrease concurrency until service times reach acceptable levels.”


| I/O Wait   | Concurrency |
| ---------- | ----------- |
| 5% o menos | 1           |
| 25%        | 5           |
| 50%        | 10          |
| 75%        | 16          |
| 90%        | 32          |
| 95%        | 64          |


## Capítulo 4: Why is My Queue So Long?

En este capítulo aprendí que hay varios factores de la infra sobre la cual se ejecuta Sidekiq que afectan a la utilización y saturación (latencia).

### Location of Saturation in Sidekiq

Arranca diciendo que idealmente encolamiento solo debe pasar en una parte en un sistema donde corre Sidekiq: en Redis. De esa forma solo sería una fuente de latencia.

> “queueing should only happen at a single location in a Sidekiq install: in the Sidekiq queues on Redis! This is important to our system design, because it means increased latency can be controlled by a single “knob”

Sin embargo, no se puede escapar de la omnipresencia de los sistemas de colas. Estos subsistemas pueden incrementar la latencia si están mal configurados. Se menciona:

**Cola de recursos del sistema**. O sea RAM y CPU. Aquí si se usa CRuby (MRI) es correr solo un proceso Sidekiq por cada vCPU/CPU que tenga la máquina.

**Cola de Redis**. Dice que:
> “the amount of transactions that a Redis database can handle per-second is proportional to the size of the keys. (...) ==a Sidekiq system with smaller arguments will scale better, because small arguments mean small Redis keys, leading to more Redis operations per second.==”

**Cola de la base de datos**. Aquí tiene que ver con el connection pool. Hay que usar [pgbouncer](https://www.pgbouncer.org/) y hay que mantener la configuración `pool` en `database.yml` igual al valor de `concurrency` de Sidekiq.

**Cola de APIs Externas**. Esto sobre es sobre ser más consciente de los límites que ofrecen las APIs de terceros para hacer un esfuerzo en encolar jobs que se van a ejecutar.

> “It would be far more efficient if we could only attempt jobs that had a possibility of succeeding.”

La clave de este es usar Sidekiq Enterprise para tener la funcionalidad de los _locks_.


## Capítulo 5: How Many Queues Should I Have?

La clave aquí es que el sistema debe tener 4-5 colas donde cada una encola trabajos que tienen una misma necesidad de tiempo de ejecución (latencia). Ejemplos de colas podrían ser: _asap_, _5_minutes_, _1_hour_, _1_day_. Donde en la cola _asap_ se encolan jobs que tengan que ejecutarse de manera inmediata.

> “most Sidekiq queues should contain jobs with similar total latency requirements. This is a fancy way of saying something like “jobs you need done right now go in the critical queue”


## Capítulo 6: Maximizing Servers without running out of resources

