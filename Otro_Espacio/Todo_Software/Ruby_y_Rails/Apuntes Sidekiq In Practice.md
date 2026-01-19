# Apuntes de Sidekiq in Practice de Nate Berkopec

## How Sidekiq Works, and Why

Sidekiq es una cola FIFO. Primero en entrar, primero en salir:
> “The threads in a Sidekiq server pull jobs from one or more queues and perform the work. They pull they oldest job from the queues they are assigned to process; Sidekiq is “first in, first out” (FIFO).”

Sidekiq opera enviando comandos a Redis para guardar las colas. La cantidad de comandos, según como se encole, importa.
> “While you don’t have to understand these commands or what they do, exactly, you should generally pay attention to the number of Redis commands used when working with Sidekiq.”

El encolado que implica dos comandos de Redis tarda más que la opción de un solo comando:
> “Sidekiq’s Redis commands are generally executed serially, so we wait for a reply from the database before sending the next command. That means that 2 commands generally take 2 times as long to execute. It also means that we’re imposing 2 times as much load on the Redis database”

Clave las opciones de encolado que implican un solo comando de Reds:
> “Because Redis is single-threaded and so can only run one command at a time, fewer Redis commands means less Redis load, which means more jobs per second.”
