# Apuntes Planning Sidekiq Queue - Judoscale

Texto: https://judoscale.com/blog/planning-sidekiq-queues

Para iniciar, lo por defecto está bien. Pero no es lo mejor.
> Could you just roll with this default setup? Sure!
>
> _Should you?_ Absolutely not! ❌

Momentos cuando nos preguntamos cómo armar las colas:
> - You have a job that needs to be run quickly, so you want it to have a higher priority than your other jobs
> - You have a job that takes a while to run, so you want it to have lower priority so it doesn’t block other jobs


## Latency is Everything

Las métricas importantes:
> - **Queue depth:** how many jobs are in a queue waiting to be processed
> - **Queue latency:** how long any given job waits in the queue before it’s processed

La más importante es Queue Latency.

## The clarity of latency-based queues

Así como dice Nate Berkopec, es mejor tener colas según el requerimiento de latencia del job:
> - “urgent” becomes “within_5_seconds”
> - “default” becomes “within_5_minutes”
> - “low” becomes “within_5_hours”

Lo importante de estas colas es que sean bien explicitas sobre la latencia esperada:
> you can change those numbers to whatever you want, and you can have more or fewer queues. The specifics don’t matter. **What matters is being explicit.**

Continua:
> We’ve encoded explicit latency expectations directly in our queue names, and by doing so we’ve avoided several problems:
>
> - When implementing a new job, choosing a queue is a _business decision_ around when the logic of that job ought to start running, not an arbitrary technical decision
> - We avoid adding unnecessary new queues because every new job will fit into an existing queue
> - We have clear performance expectations for our queues, which will guide our scaling and auto-scaling plan


## Scaling Sidekiq queues the easy way

Nota:
> Each queue’s latency should remain within its target SLA (name), using as few resources as necessary.


### Asignar colas a cada proceso

Lo ideal es que cada cola tenga su proceso. El artículo menciona que puede haber un proceso mirando todas las colas o lo anterior.

Para lo primero dicen:
> The benefit of a single process watching all of your queues is that it’s the simplest to set up, and it’s typically the most resource-efficient

Pero:
> The downsides are that you risk long-running jobs blocking other high-priority jobs, and it’s harder to autoscale.


Si se aplica lo de cada cola tiene su proceso:
> In the _dedicated process_ setup, this could never happen. Long-running jobs would only block other jobs in their own queue, which, in this case, is a low-priority queue and not a problem. High-urgency jobs would only block other high-urgency jobs (and hopefully only briefly)!

![[sidekiq.dedicated.worker.per.queue.png]]

Desventaja de un solo proceso para todas las colas:
> any time you have a single process watching more than one queue, there’s opportunity for expectation failure. It’s just not worth it!


Lo mejor es que cada cola tenga su proceso dedicado:
> In reality, the best answer here _is_ to run dedicated processes per queue. Aside from making the mental model simpler and clearer, this setup makes autoscaling a breeze. Of course, everything has trade-offs — what’s the downside to running dedicated processes? Cost


## Your recipe for Sidekiq bliss (Tl;dr:)

**How should our team structure our Sidekiq queues?**

> Your queues should be named based on the latency requirements of the jobs in those queues. Three latency-based queues (`less_than_five_seconds`, `less_than_five_minutes`, and `less_than_five_hours`) is a good starting point.


**How should I think about spreading queues across Sidekiq processes?**

> most efficient answer here is to run a dedicated process per queue. This mental model is simpler, makes autoscaling _much_ cleaner