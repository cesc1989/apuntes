# 009 - Med Requirement y PersonalReference faltantes para Therapist Bien Antiguos

Hay therapists muy viejos que no interactuaron con el form antes de pasar a la forma donde estos registros se creaban al hacer el signup. Hay que crearlos manualmente para prevenir errores.

## Query para Crear MedicareRequirement + Answers

De un solo tostazo con ActiveRecord:
```ruby
ActiveRecord::Base.transaction do
  sql = <<~SQL
    WITH new_requirements AS (
      INSERT INTO tc_medicare_requirements (tc_therapist_id, created_at, updated_at)
      SELECT tt.id, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP
      FROM tc_therapists tt
      LEFT JOIN tc_medicare_requirements tmr ON tmr.tc_therapist_id = tt.id
      WHERE tmr.id IS NULL
      RETURNING id, tc_therapist_id
    ),
    all_questions AS (
      SELECT id FROM tc_questions ORDER BY "order" ASC
    )
    INSERT INTO tc_answers (
      tc_medicare_requirement_id,
      tc_question_id,
      created_at,
      updated_at
    )
    SELECT
      nr.id,
      aq.id,
      CURRENT_TIMESTAMP,
      CURRENT_TIMESTAMP
    FROM new_requirements nr
    CROSS JOIN all_questions aq;
  SQL

  ActiveRecord::Base.connection.execute(sql)
end
```

La misma query que está dentro se puede usar en un cliente SQL.

Con estas queries puedo borrar los de Alpha para volver a probar:
```ruby
Credentialing::Answer.where(created_at: Date.current.all_day).delete_all
Credentialing::MedicareRequirement.where(created_at: Date.current.all_day).delete_all
```

## Query para Crear Personal References