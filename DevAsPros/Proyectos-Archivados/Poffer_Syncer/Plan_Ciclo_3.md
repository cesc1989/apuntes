# Plan Ciclo 3 ✅ 
Los objetivos de este nuevo ciclo con respecto a:


- OAuth de Pocket ✅ 
    - [Tener en cuenta](https://paper.dropbox.com/doc/Notas-API-de-Autenticacion-de-Pocket--BVhmP8ngYjgMT9HWEerGvA8EAQ-8lSTZ83w1ZBjg4xVquqzf)
- Primeros trabajos en gema Hootsuite ✅ 
    - [OAuth de Hootsuite](https://paper.dropbox.com/doc/Notas-API-de-Autenticacion-de-Hootsuite--BV155ZLY2GWxbJ01C4MU3UasAQ-a7yZbA1Poe2wDL0gDv8hN)
- Primeros trabajos en versión web

En esta etapa hay que completar la gema Pocket en la parte de OAuth y empezar la gema de Hootsuite o probar la 1era parte de la integración empezando el proyecto web.

## OAuth de Pocket

El tema con OAuth es que se necesita una URL callback y eso significa que se necesita un servidor web corriendo que capture el token que llega en la respuesta del API de autenticación.

## Gema Hootsuite

El trabajo aquí es todo en general. Hay que controlar el *refresh token* y también agendar publicaciones. También lleva su parte de OAuth para hacer la autenticación al API.

## Versión Web

La web es un integrador de ambas gemas. Funciona como una interfaz para ver que cosas vienen de Pocket, guardarlas en la base de datos y ver qué cosas fueron agendadas en Hootsuite.

# ¿Por dónde continuar?

Lo primero es determinar si debo hacer todo lo de OAuth en la gema Pocket. Mi duda radica en si necesito un servidor web en la gema(como hacen en la gema pocket-ruby).

Si lo necesito, igual podría copiar el de esa gema. Intentaré probar si con las pruebas automatizadas puedo obviar ese paso temporalmente mientras hago la Versión Web.

La Versión Web es la que me va a proveer de los *callbacks* para indicarselos a los servicios Pocket y Hootsuite.

Las opciones entonces son:
A) OAuth Pocket + Versión Web
B) OAuth Pocket + Gema Hootsuite(OAuth incluído)

Si queda muy complicado lo de OAuth en Pocket, empezaré la Versión Web. O sea, opción A.

