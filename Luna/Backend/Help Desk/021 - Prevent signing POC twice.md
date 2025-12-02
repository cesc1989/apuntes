# 021 - Prevent Signing POC Twice

Etiquetas: #luna_help_desk 

Caso EDG-3062

## Contexto

Pasó que un physician firmó un POC desde el Clinical Dashboard y vio un error. Al final el POC quedó resuelto pero quedó la mala experiencia.

El error:
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

# Cambios

## Arreglo Problema PlanOfCareAction

La solución es hacer que se busque o inicialicé un nuevo POCA antes de guardar. Así se evita crear un nuevo registro para el campo `plan_of_care_id` si ya hay uno previo.

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