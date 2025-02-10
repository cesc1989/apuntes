# Multi Tenancy - Apuntes Generales

Para Super Agenda, creo que montarlo en un enfoque de Multi Tenancy seria la correcto. En este documento expongo informacion al respecto de esta arquitectura de software para entender los pros y contras.

## Que es Multi Tenancy?

[Segun Wikipedia](https://es.wikipedia.org/wiki/Tenencia_m%C3%BAltiple), Multi tenancy o Tenancia Multiple es una arquitectura donde una instancia de la aplicacion se ejecuta en el servidor pero sirve a muchos clientes u organizaciones.

Agrega:

> Con una arquitectura de tenencia múltiple, la aplicación puede particionar virtualmente sus datos y su configuración para que cada cliente tenga una instancia virtual adaptada a sus requerimientos.

### Ventajas

- Permite intercambiar recursos y costos de ejecucion de una aplicacion
- Todos los datos de los clientes estan en un esquema comun y unico

### Desventajas

- Arquitectura mas compleja porque hay mas configuraciones a tener en cuenta
- La seguridad de los datos necesita un trato muy cuidado para que los datos de un cliente no se filtren a otro

>[!Note]
> Una aplicacion web, es mejor que empiece como Multi tenant (si tiene cabida). Cambiar una aplicacion normal a Multitenant es mucho mas complicado.

