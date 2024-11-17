# Los Pull Requests no son para sugerir cambios grandes en el diseño
En el mundo ideal, lo que está en el Pull Request ya se diseñó y se planeó.

Es por el título (y lo que describo a continuación) que entiendo que Jason me pida planes técnicos y spikes. Hay que entender el problema y la solución para llegar a un diseño o propuesta y el trabajo a hacer sea más claro.

Siguiendo eso el pull request se vuelve un formalismo, un espacio para compartir información, un momento para asegurarnos que se respetan reglas del proyecto y que no se incluye código duplicado.

# Contexto

Primero, este [trino](https://x.com/joeldrapper/status/1782038080003707028) de Joel Drapper (creador de Phlex)

![](https://paper-attachments.dropboxusercontent.com/s_662C2C024166A1A8042DDFD0291B4154F9A5906F712E0C89BA2833A5AD4B9DDD_1716327484024_imagen.png)


Pedir un cambio grande en un PR ya es muy tarde. Hay mucho trabajo detrás y no aceptarlo porque el diseño “no podría gustar” a quien revisa, sería perder tiempo.

Ahí la importancia de Planes Técnicos, RFCs y Spikes.

----------

También está esta [respuesta](https://x.com/yiming_dev/status/1721665162040766943) de Yiming Cheng a Ryan Singer

![](https://paper-attachments.dropboxusercontent.com/s_662C2C024166A1A8042DDFD0291B4154F9A5906F712E0C89BA2833A5AD4B9DDD_1716327605515_imagen.png)

> I used to comment a lot in the PR.

Esto me pasa bastante en mis PRs de Luna pero Ryan y Anthony proveen comentarios que no cambian drásticamente la solución. Al contrario, me guían para usar mejor lo que hay en Edge.

Este es el [trino](https://x.com/rjs/status/1716077582511169788) completo de Ryan Singer

![](https://paper-attachments.dropboxusercontent.com/s_662C2C024166A1A8042DDFD0291B4154F9A5906F712E0C89BA2833A5AD4B9DDD_1716327965490_imagen.png)


Cuando se está trabajando en algo acordado y definido es mucho más fácil agregar más cosas.

Pasa contrario cuando hay que cambiar la ruta, quitar cosas o compensar cosas.


----------

También este otro trino de Mike.

![](https://paper-attachments.dropboxusercontent.com/s_662C2C024166A1A8042DDFD0291B4154F9A5906F712E0C89BA2833A5AD4B9DDD_1716327746422_imagen.png)


Nuevamente se refuerza la idea de un plan técnico, un RFC o un Spike. Y eso eso lo que vemos en proyectos como [Ember](https://github.com/emberjs/rfcs/tree/master).


# Conclusión

Los pull requests deben ser sobre:

- Revisión pequeña
- Mucho linting automático
- Compartir conocimiento con el equipo
- Cambios menores

La clave para que sirvan así es que se hagan RFCs o planes técnicos donde se define la arquitectura. Esos sí deben ser revisados y cambiados drásticamente, si es necesario.

Escribir código debe ser el último paso en creación de software.

