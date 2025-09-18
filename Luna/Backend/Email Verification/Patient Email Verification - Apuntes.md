# Patient Email Verification - Apuntes

Desde qué partes del sistema se envía el email de verificación al paciente.

## Desde `PatientEmailUpdateWorker`

A la fecha de este doc, en línea 15:
```ruby
email_record.send_verification! if email_record.unverified?
```

línea 24:
```ruby
new_email.send_verification!
```

y línea 38:
```ruby
email_record.send_verification!
```

Se dan en la misma clase pero para diferentes ramas de un condicional.

Por Claude:

Triggers verification emails when:
  - New patient, no email record exists (line 15): `email_record.send_verification!`
  - Cash patient with changed email (line 24): `new_email.send_verification!`
  - Cash patient with unverified email (line 38): `email_record.send_verification!`

Key Rule: Only cash patients (self_pay?) require email verification

## En Admin, Patients

Archivo: `app/admin/customers/patients.rb`. Línea 405 tiene la acción que dispara la verificación.

```ruby
member_action :resend_email_verification_link, method: :post do
	email = Account.find(params[:id]).email_address

	if email.unverified?
		if email.expired_verification_code?
			email.resend_verification!
		else
			email.send_verification!
		end
		redirect_back fallback_location: admin_patient_path(resource),
									notice: "Resent verification link email!"
	else
		redirect_back fallback_location: admin_patient_path(resource),
									notice: "Patient's email has already been verified"
	end
end
```

## En Mutación `Mutations::SendVerificationEmail`

En `app/graphql/mutations/send_verification_email.rb`.

Está este bloque condicional:
```ruby
if email.expired_verification_code?
	email.resend_verification!
else
	email.send_verification!
end
```

## Admin Bulk Operations para Patients

Archivo: `app/admin/concierge/unverified_patients_report.rb:26`

Tiene esta acción:
```ruby
member_action :resend_email_verification_link, method: :patch do
	email = Account.find(params.require(:id)).email_address

	if email.unverified?
		if email.expired_verification_code?
			email.resend_verification!
		else
			email.send_verification!
		end
	else
		redirect_to request.referer, notice: "#{email.value} has already been verified"
	end
end
```