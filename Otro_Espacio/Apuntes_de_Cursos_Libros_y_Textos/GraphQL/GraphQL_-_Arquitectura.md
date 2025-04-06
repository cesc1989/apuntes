# GraphQL - Arquitectura
GraphQL tiene tres casos de uso:

1. Servidor GraphQL conectado a una base de datos
2. Servidor GraphQL como una capa por encima de servicios externos o antiguos que son integrados en un solo API
3. Servidor GraphQL híbrido. Una mezcla de los dos anteriores.

GraphQL puede lograr todo esto mediante los *Resolvers*.

Los *Resolvers* son funciones que se encargan de *resolver* los datos de cada campo de las queries o mutaciones.

La función principal de un resolver es buscar los datos de un determinado campo.

![Screenshot containing some of the resolved field names](https://imgur.com/e1gBEP5.png)


Como se ve en la imagen. Por cada campo (raíz y carga), debe haber un resolver.

