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

Encontré un paciente X para probar. Hice el trash y puedo ver esto en el log:

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