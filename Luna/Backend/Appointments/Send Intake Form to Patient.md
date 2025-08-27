# Sending Intake Form to Patient

> [!Note]
> The Intake Form is created after the care plan is created. See [[General Flow to Create Forms]]

After an "initial" appointment is created an email with the Intake Form is sent to the patient. This happens in an after create commit:
```ruby
class Appointment < ApplicationRecord
  after_create_commit :send_patient_intake_request

  # (...)

	def send_patient_intake_request
		return unless active?
		return unless visit_type == "initial"
		return unless episode.appointments.to_a.blank? || self == episode.appointments.to_a.min_by(&:created_at)
		return unless Setting.load_safe("marketplace_intake_request_disabled") == true
		return if patient.example_patient? || blacklisted? || patient.blacklisted?
	
		SendPatientIntakeRequestWorker.perform_at(15.seconds.from_now, id)
	end
end
```

En el worker `SendPatientIntakeRequestWorker` se usan los helpers de Forms para extraer la información necesaria y verificar que no esté completo. Si no está completo se usa el mailer `PatientMailer.appointment_confirmed_and_send_form` indicando el tipo de form. En este caso "intake".

```ruby
PatientMailer.appointment_confirmed_and_send_form(
	care_plan,
	appointment,
	"intake",
	intake_form_url
).deliver_now
```