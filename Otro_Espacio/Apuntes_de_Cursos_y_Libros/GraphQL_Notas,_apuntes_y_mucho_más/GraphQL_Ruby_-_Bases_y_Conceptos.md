# GraphQL Ruby - Bases y Conceptos

Documentos relacionados

- [[GraphQL_-_Arquitectura]]
- [[GraphQL_Ruby_-_En_detalle]]
- [[No me gusta de GraphQL]]

# Enlaces

- Curso Introductorio https://www.howtographql.com/graphql-ruby/0-introduction/
- GraphQL Ruby gem http://graphql-ruby.org/
- GraphiQL: IDE en el navegador para explorar servidor graphql https://github.com/graphql/graphiql


# ¿Qué es GraphQL?

Un nuevo estándar de API para proveer una alternativa más flexible con respecto a REST.

Creado por Facebook.

GraphQL permite solicitar datos de una manera declarativa. El cliente específica que datos necesita de la API. En vez de varios endpoints fijos que devuelven una estructura predeterminada, GraphQL expone un solo endpoint para responder con todo lo que pida el cliente.


## Ventajas

**Con GraphQL se reduce la cantidad de datos que se necesitan en cada petición.** Esto es ventajoso para dispositivos móviles de poca potencia o en redes de poca calidad.

**Se adapta a diferentes clientes.** GraphQL simplifica el uso del API por diferentes tipos de clientes frontend. Al final solo es un endpoint y el cliente pide los datos que necesita.

**Facilita el despliegue continuo y agiliza el desarrollo.** A diferencia de REST, en GraphQL no hay que hacer cambios puntuales en diferentes endpoints. Esto significa que un cambio en la UI no tiene que esperar a que se actualice el endpoint para devolver los nuevos datos.


# GraphQL vs REST

Algunos problemas de REST

- Tener que hacer varias peticiones para cargar los datos de una vista
- Traer información de sobra en cada endpoint
- No traer información suficiente en cada endpoint
    - Esto hace que se tengan que hacer más peticiones

¿Cómo resuelve esto GraphQL?

Debido a la naturaleza flexible de GraphQL, los devs frontend pueden implementar nuevos cambios en la UI sin necesidad de esperar modificaciones mínimas en el endpoint.

Además, GraphQL implementa un sistema de tipado para definir, a priori, un esquema de los datos y la estructura que el servidor retornará. Esto agiliza el trabajo del frontend ya que puede hacer mocks de ese tipo de estructura y una vez lanzado a los entornos conectarse al endpoint.


# Conceptos Fundamentales
## Schema Definition Language

GraphQL tiene su propio sistema de tipado para definir los diferentes schemas de la API.

Ejemplo de un tipo Person:

    type Person {
      name: String!
      age: Int!
    }

El signo de admiración significa que el campo es requerido.

Se pueden definir relaciones entre tipos:

    type Post {
      title: String!
      author: Person!
    }

Para estos caso se necesita poner la parte inversa de la relación:

    type Person {
      name: String!
      age: Int!
      posts: [Post!]!
    }


> Nota que `Post!` está dentro de un array. Esto denota una relación uno-a-muchos.


## Obteniendo datos con consultas (queries)

En GraphQL, los clientes tienen que enviar mucha más información en la petición para que el servidor entregue todos los datos deseados.

Ejemplo de query:

    {
      allPersons {
        name
      }
    }

Una query tiene dos partes:

- la raíz: `allPersons`
- la carga: `{name}`

Respuesta:

    {
      "allPersons": [
        { "name": "Johnny" },
        { "name": "Sarah" },
        { "name": "Alice" }
      ]
    }

El servidor solo retorna lo que se pida en la carga. En este caso `name`.

Este ejemplo retornaría nombre y edad:

    {
      allPersons {
        name
        age
      }
    }

De esta forma, en la misma petición, se traerían los Posts de cada persona

    {
      allPersons {
        name
        age
        posts {
          title
        }
      }
    }

**Consultas con argumentos**
Cada campo puede tener cero o más parámetros siempre que se definan en el esquema. Aquí de ejemplo se define un parámetro `last` para retornar esa cantidad de registros.

    {
      allPersons(last: 2) {
        name
      }
    }


## Escribiendo datos con mutaciones (mutations)

Las mutaciones son las formas de enviar datos al servidor para almacenar, modificar o eliminar.

Las mutaciones siguen la misma sintaxis que las consultas pero deben empezar con la palabra `mutation`:

    mutation {
      createPerson(name: "Bob", age: 36) {
        name
        age
      }
    }

Igual que las consultas:

- Hay un campo raíz `createPerson`
- El campo raíz tiene parámetros `name: , age:`
- y se puede definir una carga para retornar esos valores.

Esta sería la respuesta del servidor:

    "createPerson": {
      "name": "Bob",
      "age": 36,
    }


## Definiendo un Esquema

El esquema es una de las partes más importantes cuando se trabaja con una API GraphQL. Este define todo lo que puede hacer el API así como la forma en que los clientes pueden solicitar información.

Una forma de ver el esquema es como una colección de tipos GraphQL.

Hay algunos tipos que son root:

    type Query { ... }
    type Mutation { ... }
    type Subscription { ... }

Estos tipos son los puntos de entrada (entrypoints) para las peticiones que envían los clientes.

Por ejemplo, un esquema para los ejemplos anteriores sería:

    type Query {
      allPersons: [Person!]!
    }

Así se escribiría si se permitiera un parámetro:

    type Query {
      allPersons(last: Int): [Person!]!
    }

Así se definiría el esquema para la mutación anterior:

    type Mutation {
      createPerson(name: String!, age: Int!): Person!
    }

Así se vería el esquema completo de todo lo anterior:

    type Query {
      allPersons(last: Int): [Person!]!
      allPosts(last: Int): [Post!]!
    }
    
    type Mutation {
      createPerson(name: String!, age: Int!): Person!
      updatePerson(id: ID!, name: String!, age: String!): Person!
      deletePerson(id: ID!): Person!
    }
    
    type Person {
      id: ID!
      name: String!
      age: Int!
      posts: [Post!]!
    }
    
    type Post {
      title: String!
      author: Person!
    }

