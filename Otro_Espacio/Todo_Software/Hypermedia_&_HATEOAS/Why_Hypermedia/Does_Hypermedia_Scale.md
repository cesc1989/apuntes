# Does Hypermedia Scale?
En resumen: sí pero depende. En la mayoría de los casos, sí.


# Escalando nodos

La web en general ha escalado y la web está construida sobre Hypermedia.

Sí escala en este apartado.

# Escalando el desempeño de la aplicación

Bajo estas condiciones:

- Software should be *stateless*
- Software should support *horizontal scaling*
- Features in the software should be *independent*
- The performance of the system should be *observable*
- The software should utilize caching

Se puede ver que para cada una Hypermedia puede escalar.

Hypermedia es stateless, tal cual la describe la tesis de Roy Fielding.

En Aplicaciones Hypermedia o HDA, sus endpoints son independientes ya que retornan información al respecto de cada pantalla.

Y sobre cache:

> web applications have a long and storied history of caching. HTTP offers caching at the browser, controlled by headers. Mature server side frameworks like Rails offer sophisticated caching at the controller layer. Caching is second nature for HDAs.


# Escalando el número de características

Esto es fácilmente lograble ya que cada endpoint, en una HDA, es independiente el uno del otro según lo que tengan que ofrecer a cada pantalla.

Incluso, las vistas, mediante instrucciones del lenguaje de servidor, pueden ser reutilizadas. Esto ayuda más a lograr este escalamiento.

