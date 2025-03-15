# Race Condition aún usando find_or_initialize_by

# Enlaces
- ActiveRecord find_or_initialize_by race conditions - [Stack Overflow](https://stackoverflow.com/questions/26249956/activerecord-find-or-initialize-by-race-conditions)
- Race Conditions on Rails - [Karol Galanciak](https://karolgalanciak.com/blog/2020/06/07/race-conditions-on-rails/)


# Caso en detalle

Este documento describe este [error en Sentry](https://sentry.omega.getluna.com/share/issue/edc8ce4ee30b460994af856f394ba9bf/).

![](https://paper-attachments.dropboxusercontent.com/s_D1440438CF0497398BBBA18E34FB899F0BF0A2209C2E6EDB3A527689F84153DF_1688510851582_image.png)


El campo `email` en la tabla patients tiene una restricción única a nivel de base de datos.

En el endpoint `PUT patients/:internal_id` se busca al paciente mediante el campo `internal_id` usando el método `#find_or_initialize_by`:

    def upsertable_patient
      Patient.find_or_initialize_by(internal_id: params[:internal_id])
    end

En todo caso, pasa se da este error. ¿Por qué?

# Sospecha #1

Es una condición de carrera.

Sería en el caso de que la petición viaje dos veces. ¿Qué tan probable es esto?

¿Hay algo que deba hacerse en ese caso?

- Buscar al paciente con una combinación de `internal_id` e `email`.
- Preguntar a Ryan.


# Sospecha #2

¿Podría ser ignorancia de Marketplace sobre los registros?

Marketplace no sabe de la existencia de ese paciente mediante un `internal_id` o `email` anterior y se manda la petición.

Al no tener una referencia de algún paciente con un email definido pero con diferente `internal_id`, se envía la petición y por eso falla.

¿Hay algo que deba hacerse en este caso?

