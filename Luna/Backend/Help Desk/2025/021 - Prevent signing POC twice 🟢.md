# 021 - Prevent Signing POC Twice

Etiquetas: #luna_help_desk 

Caso EDG-3062

## Contexto

Pasó que un physician firmó un POC desde el Clinical Dashboard y vio un error. Al final el POC quedó resuelto pero quedó la mala experiencia.

El error principal:
```
PG::UniqueViolation: ERROR:  duplicate key value violates unique constraint "index_plan_of_care_actions_on_plan_of_care_id"
DETAIL:  Key (plan_of_care_id)=(c763a6db-97fb-4e1b-a085-6d1dddc49299) already exists.
```

Fabricio enlazó dos sentries. Una con el error anterior y otra con este:
```
ActiveRecord::ReadOnlyError

Write query attempted while in readonly mode: INSERT INTO "clinical_dashboard_activities" ("kind", "browser", "provider_name", "patient_name", "created_at", "updated_at", "date", "provider_id", "provider_kind", "extras", "dashboard_id", "provider_email") VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12) RETURNING "id"
```

Que se debe a que la acción para guardar las métricas de Clinical Dashboard intentó registrar una nueva en la BD de solo lectura.

# Cambios 🚧

## Arreglo Problema PlanOfCareAction

La solución es hacer que se busque o inicialice un nuevo POCA antes de guardar. Así se evita crear un nuevo registro para el campo `plan_of_care_id` si ya hay uno previo.

Las pruebas las hice con el Dr. Salya.

```
7e6fe728-a2f8-4e75-9935-cf2384999385
```

## Arreglo Problema ReadOnlyError

Aquí Claudio sugirió hacer la operación de escritura después de la respuesta HTTP.

Actual:
```ruby
head :created

event_params = JsonWebToken.decode(token: token).first.merge(sign_params)
Activity.create_sign_poc_event(event_params)
```

Propuesta:
```ruby
event_params = JsonWebToken.decode(token: token).first.merge(sign_params)
Activity.create_sign_poc_event(event_params)

head :created
```

# Pruebas en Local

Dado a que el código de Clinical Dashboard aún se relaciona con Edge mediante peticiones HTTP fue un poco más complicado para probar.

Me tocó tener dos servidores Rails. Siendo que cada "server" es una rama en un git worktree. En uno corría el código actualizado y en el otro hacía las peticiones como usuario del Clinical Dashboard.

Las clave están en las ENVs.

Con esta puedo ahorrarme el paso de verificación de token que está en Edge:
```
NEW_INFRA="true"
```

Esta es para indicar el servidor que será Edge.
```
EDGE_API_DOMAIN="http://localhost:3001"
```

Ejemplo: en la rama donde hago el cambio levanto rails normal y en el que es Edge lo levanto con:
```
bundle exec rails s -p 3001
```

Así para este caso el servidor Edge responde a las peticiones de:

- unsigned plans of care
- plan of care action

Mientras que el servidor que es el Clinical Dashboard responde a:

- generar CD link
- cargar los POCs (haciendo la petición a Edge)
- iniciar firma de POC (haciendo la petición a Edge)

