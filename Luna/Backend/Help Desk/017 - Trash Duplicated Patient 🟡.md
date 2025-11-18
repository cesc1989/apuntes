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