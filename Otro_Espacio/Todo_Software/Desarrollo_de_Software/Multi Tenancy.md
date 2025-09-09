# Multi Tenancy - Apuntes Generales

Para Super Agenda, creo que montarlo en un enfoque de Multi Tenancy seria la correcto. En este documento expongo información al respecto de esta arquitectura de software para entender los pros y contras.

## ¿Qué es Multi Tenancy?

[Según Wikipedia](https://es.wikipedia.org/wiki/Tenencia_m%C3%BAltiple), Multi tenancy o Tenancia Multiple es una arquitectura donde una instancia de la aplicación se ejecuta en el servidor pero sirve a muchos clientes u organizaciones.

Agrega:

> Con una arquitectura de tenencia múltiple, la aplicación puede particionar virtualmente sus datos y su configuración para que cada cliente tenga una instancia virtual adaptada a sus requerimientos.

### Ventajas

- Permite intercambiar recursos y costos de ejecución de una aplicación
- Todos los datos de los clientes están en un esquema común y único

### Desventajas

- Arquitectura mas compleja porque hay mas configuraciones a tener en cuenta
- La seguridad de los datos necesita un trato muy cuidado para que los datos de un cliente no se filtren a otro

>[!Note]
> Una aplicación web, es mejor que empiece como Multi tenant (si tiene cabida). Cambiar una aplicación normal a Multitenant es mucho mas complicado.

## Gemas para hacer Multitenancy en Rails

Algunas librerías populares y su estado actual.

**Acts as Tenants**

Sitio web -> https://github.com/ErwinM/acts_as_tenant 

Ultimo commit: 6 de Septiembre de 2024.

**Apartment**

Sitio web -> https://github.com/influitive/apartment

Ultimo commit: 25 de Junio de 2019.

🌟 **ros-apartment** 🌟

> Este es un fork de apartment el cual si recibe mantenimiento.

Sitio web -> https://github.com/rails-on-services/apartment

Ultimo commit: 9 de Enero, 2025.

# Vale la pena montar una web app con arquitectura Multi tenant?

Por coincidencia, en el subreddit de ruby on rails [alguien hizo una pregunta pertinente](https://www.reddit.com/r/rails/comments/1ik7caq/multitenancy_vs_multi_instances/?utm_name=mweb3xcss) a este tema. Las respuestas las recopilo aquí:

El usuario pregunta si es mejor una arquitectura multi tenant o mejor que cada cliente este en su propio servidor y base de datos.

**Ambos**

Para la mayoría de clientes, compartir recursos estará bien. Aquí usa Multitenant.

Si un cliente pide separar sus datos o mas capacidad, haz una instalación personalizada que costar mas de lo normal.

**Multi tenancy a menos que puedas cobrar mucho mas**

Mantener instancias de muchos clientes por separado es mas desgastante y laborioso.

**Multi tenancy sin pensarlo**

Despliegues de instancias por cliente significa mas backups, analitica, herramientas, logs, seguridad por cada uno. Mucho trabajo. Esto solo tiene sentido si se cobra mucha plata.

# Formas de Multi Tenancy

Hay tres formas claras:

- Simple: llave foranea solamente.
- Row-level: tenant_id en cada tabla
- Schema: usando esquemas de la base de datos para separar las mismas tablas para cada cliente

En la forma simple es como hice Bucket. El padre de todos los datos es el registro usuario o account.

En la forma row-level cada tabla tiene un tenant_id para diferenciar los registros. Essta es la forma en que funciona la gema acts-as-tenant.

En la forma Schema se usa la caracteristica de los esquemas de la base de datos para crear las tablas en diferentes esquemas segun haya clientes en el negocio. Esta es la forma en que funciona la gema ros-apartment.