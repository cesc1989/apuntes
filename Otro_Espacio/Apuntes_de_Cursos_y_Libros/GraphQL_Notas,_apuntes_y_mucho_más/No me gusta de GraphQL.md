# Cosas que no me gustan de GraphQL

Al ver la forma en que se usa en Edge noto varias cosas que no calan, desde mi falta de experiencia, sobre esta herramienta.

## Tipos

Una enorme cantidad de archivos de tipos. Peor cuando esos archivos de tipos van más allá de usar las macros que define la gema.

## Pruebas

Hay unas pruebas donde toca escribir toda la query en un HEREDOC enorme. Eso no se ve en APIs REST.

Aparte de los Heredocs enormes que contienen toda la query graphql, también muchos examples de rspec de muchísimas líneas. Cosas locas que no veo normalmente en las pruebas de endpoints REST.

![[large.spec.for.graphql.png]]

No sé si en otras aplicaciones de mayor tamaño que no usen GraphQL se vean cosas así pero esto que veo no me parece bueno.

## Resolvers que hacen mil y una cosas

Resolvers enormes.

## Pruebas para medir del desempeño del endpoint GraphQL

Esto basicamente:
```
./spec/requests/graphql/performance/production_queries_spec.rb
```

Eso no lo he visto nunca en APIs REST.

## GraphQL Pro y Enterprise

Esto https://graphql.pro/ y cuesta 900 usd/año. O esto https://graphql.pro/enterprise que cuesta 1900 usd/año. ¡¿Qué?!

O sea, no existe REST Pro o REST Enterprise.

Características clave de GraphQL Ent:
- rate limiting
	- En REST se usa rack-attack
- server-side caching
	- Rails lo ofrece por defecto?
	- Cache estándar???
