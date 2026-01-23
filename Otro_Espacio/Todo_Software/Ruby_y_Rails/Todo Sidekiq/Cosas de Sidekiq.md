# Apuntes Variados de Sidekiq y Cómo Funciona

Etiquetas: #sidekiq 

## Sobre la latencia

He visto esto en Edge Alpha:

- (hace 9 minutos) 586.05
- (hace 9 minutos) 579.66

Según Gepeto eso no es positivo:
> En Sidekiq, **la latencia** es **cuánto tiempo lleva esperando el job más viejo en la cola antes de empezar a ejecutarse**.
> significa:
> 
>  - Hay al menos **un job** en la cola `default`
>  - Ese job fue **encolado hace ~586 segundos** (~9 minutos)
>  - **Sidekiq todavía no lo ha empezado a procesar**
>
> No es duración del job.  
> No es tiempo promedio.  
> Es **tiempo de espera**.


