# 020 - Resolver duplicados de email en tc_therapists

Etiquetas: #luna_help_desk 

Caso EDG-3076.

## Contexto

La tabla `therapists` en therapist-credentialing-backend si bien el campo `email` tuvo índice único este no trabajaba de la forma 100% correcta porque un email podría estar dos veces en la tabla siempre que tuviera diferente casing.

Ejemplo:
```
test@example.com
TEST@example.com
```

La migración original es esta:
```ruby
t.index ["email"], name: "index_tc_therapists_on_email", unique: true
```

Lo anterior causó que se pudieran registrar un montón de correos duplicados cuya única diferencia era si estaban en mayúsculas o minúsculas.

Como parte de los esfuerzos de AppBlend, al pasar esta tabla a `backend` había que desactivar el índice único en dicha base de datos. Una vez con los datos pasados por Logical Replication e intentar activar el índice habría error porque habría duplicados.

## Solución

Para corregir esta situación me tocó ubicar todos los duplicados, elegir el candidato para "desduplicar", cambiarle su correo y luego agregar la migración de índice único.

### Query para encontrar los duplicados

Con esta query encontré todos los duplicados:
```sql
SELECT
  t.id,
  t.first_name,
  t.last_name,
  t.email,
  t.hubspot_id,
  t.created_at,
  t.updated_at
FROM tc_therapists t
WHERE LOWER(t.email) IN (
  SELECT LOWER(email)
  FROM tc_therapists
  WHERE email IS NOT NULL
  GROUP BY LOWER(email)
  HAVING COUNT(*) > 1
)
ORDER BY LOWER(email);
```

### Script para desduplicar

Este es el script final que recibe una lista de ids de `tc_therapists` para luego cambiarles su correo y quitarles el `hubspot_id`:
```ruby
therapists = Credentialing::Therapist.where(id: dups)
therapists.each do |therapist|
  email = therapist.email

  therapist.update_columns_with_audit(
    email: "dupped_#{email}",
    hubspot_id: nil,
    audit_comment: "Duplicated Therapist record"
  )
end
```

