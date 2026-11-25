# Detalles de los endpoints para publicar/programar

> [!Important]
> Así están estos endpoints a la fecha 25/11/2026.


## Relación endpoints y funcionalidad

| Intención del usuario | Llamadas | Endpoint |
|---|---|---|
| Guardar borrador | 1 | `POST /api/v1/posts` sin `scheduled_at` → `draft` |
| **Crear y programar** | **1** | `POST /api/v1/posts` con `scheduled_at` → `scheduled` |
| Crear y publicar ya | 2 | `POST /posts` (crea el draft) y luego `POST /posts/:id/publish_now` |
| Programar o reprogramar un post existente (`draft`, `scheduled` o `failed`) | 1 | `POST /posts/:id/schedule` |
| Desprogramar | 1 | `DELETE /posts/:id/schedule` → vuelve a `draft` |


### ¿Por qué crear y programar hace dos llamados?

Se podría simplificar pero se volvería un _merecumbé_. Habría que gestionar diferentes respuestas. Tanto del API como del provider.

De esta forma (con dos llamados) si el post falla en ser posteado, al menos va a quedar en `draft`. O sea, existente en la base de dato. Así se da cabida para avisar al usuario y que tome acciones.

## Formato de fecha para programar post

Además de ser una fecha en el futuro, debe ser así:
```ruby
"2026-11-23T17:00:00-05:00"
```