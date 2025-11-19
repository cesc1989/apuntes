# 017 - Trash Duplicated Patients fails

Etiquetas: #luna_help_desk 

Caso EDG-3028

Un usuario intentó "trash" un paciente duplicado y no pasó nada al final.

## Contexto

Esta característica está disponible en el formulario de editar paciente. Es un checkbox con etiqueta "Trash Record".

Para poder usar el checkbox se deben cumplir las condiciones:
```ruby
<% if @patient.persisted? && ((@patient.whitelisted? && @patient.can_blacklist?) || (@patient.blacklisted? && @patient.can_whitelist?)) %>
```

Esos métodos están definidos en el modelo Account:
```ruby
def whitelisted?
	!blacklisted?
end

def can_blacklist?
	return false if typed_user.blank?

	typed_user.appointments.select(&:active?).blank?
end

def can_whitelist?
	return true if typed_user.blank?
	return true if therapist?

	!typed_user.example_patient?
end
```

Importante es `can_blacklist?` que revisa si el paciente tiene appointments activos. Ver [[Edge Enums que Importan 🔑#Appointment]]

El "trash" lo que hace es cambiar los valores del paciente para que no se confunda en otras partes del sistema. Eso se puede ver en `BlacklistAccountsService`.

## Datos

Paciente duplicado: Snyder.
ID a trashear: `02cd19d3-b1b5-46ce-9f67-3fc33d8852c9`
Email a trashear: `9512276379@call.com`

El paciente tiene un care plan que no tiene appointments activos así que puede ser trasheado.

# Problema #1

Hay una operación en el proceso de actualizar el account a trash. Se hace un rollback en un punto y no se completa. Como el job `PatientEmailUpdateWorker` corre de manera asincrona se crea el registro en `user_communication_methods` pero se dejó el account sin trashear.

Entonces cuando se intenta de nuevo revienta con lo que se ve en el Problema #2 ya que intenta crear de nuevo el `user_communication_methods` con el mismo email del intento anterior.

## ¿Qué causa el rollback?

Así lo vi probando en local:
```bash
Episode Load (1.0ms)  SELECT "episodes".* FROM "episodes" WHERE "episodes"."patient_id" = $1 AND "episodes"."status" = $2 ORDER BY "episodes"."created_at" DESC LIMIT $3  [["patient_id", "f9da6210-cfe6-4796-af65-9710acfd63e9"], ["status", 666], ["LIMIT", 1]]
  ↳ app/models/patient.rb:970:in `recent_draft_care_plan'
  Episode Load (0.4ms)  SELECT "episodes".* FROM "episodes" WHERE 
  
"episodes"."status" != $1 AND "episodes"."patient_id" = $2 ORDER BY "episodes"."created_at" DESC LIMIT $3  [["status", 666], ["patient_id", "f9da6210-cfe6-4796-af65-9710acfd63e9"], ["LIMIT", 1]]
  
  ↳ app/models/patient.rb:957:in `recent_care_plan'
  Clinic Load (0.3ms)  SELECT "clinics".* FROM "clinics" WHERE "clinics"."id" = $1 LIMIT $2  [["id", "6f49981a-7725-48a2-b307-ee684d31ba3f"], ["LIMIT", 1]]
  ↳ app/models/patient.rb:1337:in `validate_region_integrity'

  TRANSACTION (1.0ms)  ROLLBACK
  ↳ app/admin/customers/patients.rb:2514:in `update'
```

Cuando se actualiza el paciente se correr la validación `validate_region_integrity`. Este revisa que la nueva región incluya a la clinic del care plan.

En el caso del reporte el paciente tiene la clinic `luna_care_orange_county` pero la región Out of Area no incluye a este (no incluye a ninguna).

Por ello la validación falla y se hace el rollback.

## Solución #1 🟡

Evitar la validación `validate_region_integrity` si se está trasheando el registro. La solución luce sencilla, sin embargo, ha dado problemas porque la validación se correr tres veces y en la segunda iteración falla el guard.

```ruby
return if is_moving_to_out_of_area && is_being_blacklisted
```

Ejemplo de logs:
```
validate_region_integrity: region_id_changed=true, blacklisted=true, region_id=3, out_of_area_id=3
validate_region_integrity: region_id_changed=true, blacklisted=false, region_id=3, out_of_area_id=3
validate_region_integrity: region_id_changed=true, blacklisted=true, region_id=3, out_of_area_id=3
```

También se ve en esto:
```
=== validate_region_integrity DEBUG ===
  region_id_changed?: true
  region_id: 3, out_of_area_id: 3
  is_moving_to_out_of_area: true
  account.blacklisted?: true
  account.blacklisted: true
  account.blacklisted_changed?: true
  is_being_blacklisted: true
  GUARD RESULT: true
=== validate_region_integrity DEBUG ===
  region_id_changed?: true
  region_id: 3, out_of_area_id: 3
  is_moving_to_out_of_area: true
  account.blacklisted?: false
  account.blacklisted: false
  account.blacklisted_changed?: false
  is_being_blacklisted: false
  GUARD RESULT: false
=== validate_region_integrity DEBUG ===
  region_id_changed?: true
  region_id: 3, out_of_area_id: 3
  is_moving_to_out_of_area: true
  account.blacklisted?: true
  account.blacklisted: true
  account.blacklisted_changed?: true
  is_being_blacklisted: true
  GUARD RESULT: true
```

La solución que Claudio sugirió es saltar esta validación si el paciente duplicado se está moviendo a Out of Area:
```ruby
out_of_area_region_id = Region.find_by(name: "out_of_area")&.id
return if region_id_changed? && region_id == out_of_area_region_id
```

### ¿Es OutOfArea solo para Trashing? 👍🏽

Parece que sí. Corrí esta query contra producción:
```sql
SELECT
  a.id as account_id,
  a.email,
  a.user_type,
  a.blacklisted,
  a.region_id,
  r.name as region_name,
  r.label as region_label,
  a.first_name,
  a.last_name,
  a.created_at
FROM accounts a
JOIN regions r ON r.id = a.region_id
WHERE a.email LIKE 'trash-record+%@getluna.com'
  AND r.name != 'out_of_area'
ORDER BY a.created_at DESC
LIMIT 20;
```

Y solo devolvió tres registros con correo tipo trash que no están en OutOfArea.

# Problema #2

Para este caso parece estar en una tabla relacionada. Este es el mensaje del sentry:
```
PG::UniqueViolation: ERROR:  duplicate key value violates unique constraint "idx_user_communication_methods_kind_val_uniq"
DETAIL:  Key (user_type, user_id, kind, value)=(Account, 02cd19d3-b1b5-46ce-9f67-3fc33d8852c9, 0, trash-record+patient-02cd19d3-b1b5-46ce-9f67-3fc33d8852c9@getluna.com) already exists.
```

Claudio me dio una query para verificar si ese email trash existe en la tabla `user_communication_methods`:
```sql
-- Check if trash email exists in user_communication_methods
SELECT
  id,
  user_type,
  user_id,
  kind,
  value as email,
  created_at,
  updated_at
FROM user_communication_methods
WHERE value = 'trash-record+patient-02cd19d3-b1b5-46ce-9f67-3fc33d8852c9@getluna.com';
```

Y en efecto existe:

- kind: 0
- email: `trash-record+patient-02cd19d3-b1b5-46ce-9f67-3fc33d8852c9@getluna.com`
- created: `2025-11-06 17:21:55.646`

## ¿Por qué llega hasta UserCommunicationMethod?

Cuando se completa un update desde el admin en Luxe se encola el worker `PatientEmailUpdateWorker`. En este hay una verificación del correo. Si hay diferencia se procede a actualizar y luego se procede a crear un nuevo registro en dicho modelo con:
```ruby
UserCommunicationMethod.email.create!(email_attributes)
```

En este caso ya existe el email en la versión trash por eso cuando llega a este paso falla el proceso.

# Pruebas en Local

## Replica

Encontré un paciente X para probar con esta query:
```sql
SELECT
  p.id as patient_id,
  a.id as account_id,
  a.email as current_email,
  a.first_name,
  a.last_name,
  a.blacklisted,
  ucm.value as communication_email,
  ucm.created_at as comm_method_created,
  COUNT(DISTINCT apt.id) FILTER (WHERE apt.state IN (0, 1, 2)) as active_appointments,
  COUNT(DISTINCT apt.id) as total_appointments
FROM patients p
JOIN accounts a ON a.id = p.account_id
JOIN user_communication_methods ucm ON ucm.user_id::uuid = a.id
LEFT JOIN episodes e ON e.patient_id = p.id
LEFT JOIN appointments apt ON apt.episode_id = e.id
WHERE ucm.user_type = 'Account'
  AND ucm.kind = 0  -- email type
  AND a.blacklisted = false  -- not currently trashed
  AND a.email NOT LIKE 'trash-record+%'  -- not a trash email
  AND a.email NOT LIKE '%@getluna.com'  -- not internal Luna email
  -- Avoid real patients - look for test-like names
  AND (
    a.first_name ILIKE '%test%' OR
    a.last_name ILIKE '%test%' OR
    a.first_name ILIKE '%duplicate%' OR
    a.last_name ILIKE '%duplicate%' OR
    a.email ILIKE '%test%'
  )
GROUP BY p.id, a.id, a.email, a.first_name, a.last_name, a.blacklisted, ucm.value, ucm.created_at
HAVING COUNT(DISTINCT apt.id) FILTER (WHERE apt.state IN (0, 1, 2)) = 0
ORDER BY p.created_at DESC
LIMIT 10;
```


Hice el trash y puedo ver esto en el log:

En el servidor:

- La recepción de los parámetros
	- Se ve el nuevo email en modo trash
		- `"email"=>"trash-record+patient-f9da6210-cfe6-4796-af65-9710acfd63e9@getluna.com"`
- Se crea un nuevo registro en audits
- Se actualiza `patient_identity_block_group_id`

```
UPDATE "patients" SET "patient_identity_block_group_id" = $1 WHERE "patients"."id" = $2  [["patient_identity_block_group_id", "eae718c8-c0da-4457-93ba-88645b62c8ea"], ["id", "f9da6210-cfe6-4796-af65-9710acfd63e9"]]
```

- un rollback
- finalmente el redirect al perfil del paciente `Redirected to http://localhost:3000/admin/patients/f9da6210-cfe6-4796-af65-9710acfd63e9`

Cuando recargo no veo que aparezcan los valores Trash.

En el job:

- Se actualiza `user_communication_methods`
```
UPDATE "user_communication_methods" SET "primary_method" = $1, "updated_at" = $2 WHERE "user_communication_methods"."id" = $3  [["primary_method", false], ["updated_at", "2025-11-18 21:50:16.431469"], ["id", "66b95a16-7a02-4ba4-bd4e-5ece457e86b5"]]
```

- se crea un nuevo record en `user_communication_methods` con el trash email
	- `["value", "trash-record+patient-f9da6210-cfe6-4796-af65-9710acfd63e9@getluna.com"]`