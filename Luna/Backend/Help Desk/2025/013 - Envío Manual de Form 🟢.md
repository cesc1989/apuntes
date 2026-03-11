# 013 - Envío Manual de Form de Care Plan Inactivo

Etiquetas: #luna_help_desk 

Caso EDG-2923

No se puede enviar un Progress Form de un care plan inactivo porque la UI solo lo permite para los que están activos.

## Context

En el perfil del paciente, cuando el care plan actual está activo hay dos enlaces para forzar el envío del Intake o Progress form que hacen una petición a estas acciones definidas en `app/admin/customers/patients.rb`:
```ruby
# Send the "Resending the Intake Form" email strictly for the patient to receive the link.
member_action :resend_intake_form, method: :post do
	PatientMailer.resend_intake_form(resource, params[:intake_form_url]).deliver_now

	redirect_back fallback_location: admin_patient_path(resource),
								notice: "Sent schedule reminder email w/ link to intake form!"
end

# Send the "Resending the Progress Form" email strictly for the patient to receive the link.
member_action :resend_progress_form, method: :post do
	resource.send_progress_form_reminders({
		form_url: params[:progress_form_url],
		reminder_type: "resend",
		form_type: "progress"
	})

	redirect_back fallback_location: admin_patient_path(resource),
								notice: "Sent schedule reminder w/ link to progress form!"
end
```


Los enlaces están en `app/views/admin/patients/_actions.html.erb` y se muestran si se cumple la condición de que el form del care plan más reciente no se haya completado.

En el caso de este reporte como el paciente tiene dos care plans los botones no aparecen porque el Progress que les interesa es del care plan inactivo.

## Solución: Envío Manual

Hacer una prod-op para enviar de manera manual el Progress Form del care plan inactivo.

Ejemplo para un caso en Alpha:
```ruby
patient = Patient.find("0648477f-bcdc-4c68-8b3b-03c74e121373")
form = PatientSelfReport::Form.find_by(uuid: "c311c2dc-ac39-402b-88a8-134318db6fe4")
form_url = form.web_url_v3

PatientMailer.resend_progress_form(patient, form_url).deliver_now
```