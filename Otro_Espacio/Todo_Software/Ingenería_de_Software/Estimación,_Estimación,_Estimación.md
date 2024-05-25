# Estimación, Estimación, Estimación
TL;DR(Muy largo; No leí)

Las estimaciones no son para tomarse como última palabra, firmadas en sangre, un juicio final. Deben ser vistas como una guía para conocer el rumbo de un proyecto.

Deben ser creadas en entornos donde haya suficiente información para realizarlas, con el suficiente espacio y análisis para intentar sacar a luz los imprevistos(que siempre los habrá).

Aceptar la incertidumbre que existe al crear software. Todos los involucrados deben saber que existe y aceptarla. Cuando esto ocurre saldrán soluciones para abordar los problemas a encontrar al desarrollar un software.

Cuando se aceptan las estimaciones como algo concreto, que no está expuesto a factores externos que afecten el normal de desarrollo todos salen perjudicados. **Pero quien más sale perjudicado es el desarrollador**:


- Cliente frustrado porque no se entregó X a tiempo que se dijo
- Jefe frustrado porque no se están cumpliendo los tiempos y el cliente demuestra el descontento
- Gestor de proyectos frustrado porque no le están cumpliendo los tiempos y el jefe y el cliente le demuestran su descontento
- Desarrollador cansado de trabajar horas extras, tener que enfocarse en cantidad y no en calidad, gestor de proyectos y jefe encima demostrando el descontento al no estimar bien ni cumplir lo indicado.
## Lista de artículos
- [Why Asking Software Developers for Time Estimates Is a Terrible Idea and How to Bypass It](https://www.romenrg.com/blog/2015/09/28/why-asking-developers-for-time-estimates-in-software-projects-is-a-terrible-idea-and-how-to-bypass-it-with-scrum/)
- [Coding, Fast and Slow: Developers and the Psychology of Overconfidence](http://blog.hut8labs.com/coding-fast-and-slow.html)
- [No Deadlines For You! Software Dev Without Estimates, Specs or Other Lies](http://blog.hut8labs.com/no-deadlines-for-you.html)
- [Software Estimation is a Losing GameShould we even bother?](https://rclayton.silvrback.com/software-estimation-is-a-losing-game)
- [Why We Don't Do Fixed-Price Software Projects (And Neither Should You)](https://blog.salsitasoft.com/why-we-dont-do-fixed-price-software-projects/)
- [Why are software development task estimations regularly off by a factor of 2-3?](https://qr.ae/TW88bL)
- [Why Estimation in Software Development is Usually Terrible](https://m.coderdan.co/why-estimation-in-software-development-is-usually-terrible-2d032161bc6)
- [Why Software Projects Tend to Fail](https://www.codeproject.com/Articles/20488/Why-Software-Projects-Tend-to-Fail)
- [The Cone of Uncertainty(este es del libro Software Estimation de Steve McConell](https://www.construx.com/software-thought-leadership/books/the-cone-of-uncertainty/)
- [How to estimate in software teams — Part 1](https://m.coderdan.co/how-to-estimate-in-software-teams-71c6b4a799b1)
- [The Hill chart for estimating tasks](https://m.signalvnoise.com/new-in-basecamp-see-where-projects-really-stand-with-the-hill-chart-ca5a6c47e987)
- [Why Fixed Bids Are Bad For Clients, Too](https://thoughtbot.com/blog/why-fixed-bids-are-bad-for-clients-too)
- [Planning is Guessing](https://m.signalvnoise.com/planning-is-guessing/)
- [Artículo y comentarios: How do you estimate time required for assigned task?](https://dev.to/imben1109/how-do-you-estimate-time-required-for-assigned-task-57md)
- [No ETAs](https://inessential.com/2019/10/28/no_etas)
# Resumen General
## Coding, Fast and Slow: Developers and the Psychology of Overconfidence

Artículo: http://blog.hut8labs.com/coding-fast-and-slow.html

**Resumen**
Estimamos mal porque por una parte, muchas veces, desconocemos las implicaciones no visibles de una nueva tarea o requerimiento y en otra parte porque somos demasiado confiados al estimar.

Según el libro *Thinking Fast and Slow* de Daniel Kahneman, hay dos sistemas. **Sistema I y Sistema II**.

**Sistema I** se encarga de hacer análisis rápido de información mediante patrones previamente vividos.
**Sistema II** se encarga de hacer análisis concienzudo de la información a una velocidad mucho menor.

Ambos conviven para que como seres humanos podamos sobrevivir pero a la hora de estimar, generalmente, **prevalece el Sistema I**. Y lo hace porque generalmente se nos piden estimados para responder en cuestión de segundos, no horas. 

**¿Solución?**

Ser conscientes de que somos demasiado confiados al estimar, aceptar lo cambiante y la incertidumbre que hay al realizar software y adoptar mecanismos para registrar y analizar estimaciones pasadas.

Llevar registro y control de estimaciones, no para decidir una fecha de entrega o saber si el proyecto está atrasado sino para mejorar al estimar. Así se puede entrenar el Sistema I para que esté más afinado cada que intervenga en estimaciones.

La solución **NO** es sentarse una semana a planificar y estimar todas las tareas de un proyecto. La solución es hacer el ejercicio constante y retroalimentar el proceso a plazos cortos.


## No Deadlines For You! Software Dev Without Estimates, Specs or Other Lies

Artículo: http://blog.hut8labs.com/no-deadlines-for-you.html

**Resumen**
Cuando se pide un estimado para una determinada tarea o requerimiento, hay dos formas de reportar la solicitud:


- Aceptar la intervención del Sistema I y dar un estimado sin analizar nada.(peor de los casos)
- Solicitar el espacio y más información para sacar una mejor conclusión de la tarea a hacer y dar un mejor estimado(aunque también pueda ser erróneo)

Nuestro deber como desarrolladores es entender los problemas para los cuales nos piden un requerimiento o desarrollo para poder ofrecer un buen estimado o también ser capaces de proponer mejores soluciones.

Al entender el problema también podemos entrar a negociar. Y lo podemos hacer al buscar una forma de ir entregando lo que se pide de forma parcial pero de tal forma que vaya tomando la forma real y además ir supliendo la necesidad existente.

## Software Estimation is a Losing Game

Artículo: https://rclayton.silvrback.com/software-estimation-is-a-losing-game

**Resumen**
Cuando aceptamos una fecha límite, bajo estimados que muy posiblemente están mal elaborados, puede ocurrir lo siguiente:


- **Perdida de credibilidad** con el cliente y el equipo
- **Fricción interna** en el equipo cuando los gestores prometen demás
- **Baja moral** al equipo tener que trabajar demás(extras, etc)
- **Software de poca calidad** al saltarse procesos de tests o aplicación de mejores prácticas con tal de entregar
- **Muerte del proyecto** cuando el cliente se cansa de los atrasos o de pagar demás, si es el caso.

El problema real con la estimación no es hacerla sino que somos muy malos como desarrolladores al estimar.

Pero lo somos al tratar de estimar términos muy extensos. Una estimación será más precisa en la medida en que el tamaño y tiempo de lo que se pida sea reducido.


- ¿Cuánto tardas en hacer una aplicación cómo Uber? - No sé, 1 año. **MAL**.
- ¿Cuánto tardas en implementar la API de PayU en mi software? - Tal vez 1 o 2 semanas. **UN POCO MEJOR**.

Además, un estimado empieza a incumplirse cuando durante el período de trabajo, sea un sprint o lo que sea, se ingresan más tareas o cuando algo estaba mal definido.

Y que algo estuviera mal definido pudo haber evitado o no. Hay mucha incertidumbre al hacer software y no podemos cubrir todos los imprevistos. A veces hay que aceptarlos y como hay que aceptarlos no podemos pensar que un estimado está firmado en piedra porque obviamente será afectado.

**¿Solución?**


- No ofrecer contratos fijos. Son propensos a fallas al estimar y sobre costos.
- Darle al cliente la facilidad de reorganizar las tareas, agregar y/o quitar durante todo el ciclo de vida.
- Interacción y retroalimentación constante del cliente en cada etapa.

Tal cual como lo indican las metodologías ágiles pero tomando los estimados como una pizca de sal.


## Why are software development task estimations regularly off by a factor of 2-3?

Ver respuesta en Quora: https://www.quora.com/Why-are-software-development-task-estimations-regularly-off-by-a-factor-of-2-3/answer/Michael-Wolfe?ch=10&share=a82e31c9&srid=dMUc


## Why Estimation in Software Development is Usually Terrible

Artículo: https://m.coderdan.co/why-estimation-in-software-development-is-usually-terrible-2d032161bc6

**Resumen**
Somos malos estimando por lo que los psicólogos describirían como el efecto *Dunning-Kruger.* Los más inexpertos tienden a sobre estimar sus habilidades mientras que los más expertos hacen lo opuesto.

![](https://cdn-images-1.medium.com/max/1600/1*7HPFeJPLKJe0s581xjsyJw.jpeg)


El problema con las estimaciones equivocadas es la incertidumbre. Mientras menos certeza tengamos de cómo hacer algo más pobres serán nuestros estimados para saber cuánto tiempo nos tomará completarlo.

Las ambigüedades al definir una tarea también afectan los estimados. Lo que pide el cliente puede ser muy diferente a lo que tenemos en mente para esa petición.

Un consenso claro entre las partes y usar herramientas visuales para unificar los requerimientos sirven mucho a la hora de estimar.

