# Example Patients

¿Cómo los encuentro? Hay dos formas. Buscando por prefijo o por el email.

## Prefijo: `luna-example-patient-`

Como se ve en
```ruby
# app/admin/customers/therapist.rb

account = Account.find_or_create_by!(email: "luna-example-patient-#{resource.id}@example.com") do |new_account|
	set_example_patient_account_data(new_account)
end
```


```ruby
# app/views/admin/therapists/_info.html.erb

example_patient_account.email = "luna-example-patient-#{resource.id}@example.com"
```

```ruby
# spec/factories/accounts.rb

factory :example_patient_account, parent: :account do
	first_name { "Example" }
	email { "luna-example-patient-#{Random.rand}@example.com" }
	blacklisted { true }
end
```

## Correo: `example.com`

Muy bien se puede ver en los ejemplos anteriores que igual que el prefijo tienen el correo `example.com`.

Ejemplo:
```ruby
# app/workers/cancel_example_patient_appointments_worker.rb

example_patients = Patient
	.includes(:account, episodes: :appointments)
	.joins(:account)
	.where("accounts.email ILIKE '%example.com'")
	.joins(episodes: :appointments)
	.where(appointments: { scheduled_date: ..expire_before })
	.where.not(appointments: { emr_state: "cancelled_by_admin" })
```

