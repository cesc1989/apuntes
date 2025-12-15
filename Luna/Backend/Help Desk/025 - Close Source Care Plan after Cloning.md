# 025 - Close Source Care Plan after Cloning

Etiquetas: #luna_help_desk 

Caso EDG-3029

Se quiere que al clonar un care plan, el original se cierre (con `auto_discharged`) si no tiene citas de ningún tipo.

## Contexto

La clonación se puede hacer desde el perfil del paciente. En la sección de Care Plans cada uno tiene un enlace "Clone". Al clicar, nos vamos a un formulario de nuevo care plan que carga los datos del original.

## Pruebas

### Sin appts cerrar

Necesito encontrar care plans sin appointments. Para ello tengo que usar esta query:
```ruby
pats = Patient.where(
  Episode
  .unscope(where: :status)
  .where("episodes.patient_id = patients.id")
  .where(status: :active)
  .where.not(payer_id: nil)
  .where.not(Appointment.where("appointments.episode_id = episodes.id").select(1).arel.exists)
  .select(1).arel.exists
).limit(3)
ap pats.pluck(:id)
```

Esa query se traduce en esta con subqueries:
```sql
SELECT patients.*
FROM patients
WHERE EXISTS (
	SELECT 1
	FROM episodes
	WHERE episodes.patient_id = patients.id
		AND episodes.status = 0  -- 0 = active enum value
		AND episodes.payer_id IS NOT NULL
		AND NOT EXISTS (
			SELECT 1
			FROM appointments
			WHERE appointments.episode_id = episodes.id
		)
)
LIMIT 3;
```

### No cerrar si tiene appts

Para probar el caso donde el care plan tiene appointments uso esta variación de la query:
```ruby
pats = Patient.where(
  Episode
  .unscope(where: :status)
  .where("episodes.patient_id = patients.id")
  .where(status: :active)
  .where.not(payer_id: nil)
  .where(
    Appointment
    .where("appointments.episode_id = episodes.id")
    .select(1).arel.exists
  )
  .select(1).arel.exists
).limit(3)
ap pats.pluck(:id)
```