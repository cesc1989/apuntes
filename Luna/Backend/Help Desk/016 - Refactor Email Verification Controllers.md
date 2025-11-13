# 016 - Refactor Email Verification Controllers

Etiquetas: #luna_help_desk 

Caso EDG 2743

Reporte

A causa de un error donde pacientes estaban siendo redireccionados a la página de success de verificación de email para los usuarios de Clinical Dashboard Fabricio optó, para arreglarlo, por duplicar el controlador. La diferencia terminó siendo mínima pero fue necesario para arreglar el lío.

Como parte de las tareas salidas del Postmortem está refactorizar estos controladores para simplificar el código y limpiar lo que se hizo para dejar todo en mejor forma.

## Contexto

Los controladores son:

- `app/controllers/user_communication_methods/email_verifications_controller.rb`
- `app/controllers/user_communication_methods/patient_email_verifications_controller.rb`

Y las diferencias son:

En `EmailVerificationsController` en la acción `new` la redirección:
```ruby
if @communication_method&.verified?
	redirect_to verify_email_success_path(base64_token: params.require(:base64_token))
end
```

En cambio en `PatientEmailVerificationsController` en la misma acción la redirección es:
```ruby
if @communication_method&.verified?
	redirect_to patient_verify_email_success_path(base64_token: params.require(:base64_token))
end
```

Los métodos `success` también tienen sus diferencias.

En `EmailVerificationsController`:
```ruby
def success
	token_hash = UserCommunicationMethods::Email.decode_url_token(params.require(:base64_token))
	id = token_hash.delete(:id)

	@ucm = UserCommunicationMethod.includes(:user).find_by(id: id) if id.present?

	@cd_link =
		if @ucm.user.respond_to?(:send_portal_access_link)
			generate_clinical_dashboard_access_link_for(@ucm.user, @ucm.value)
		elsif @ucm.user.respond_to?(:parent) && @ucm.user.parent.respond_to?(:send_portal_access_link)
			generate_clinical_dashboard_access_link_for(@ucm.user.parent, @ucm.value)
		end
end
```

En cambio en `PatientEmailVerificationsController`:
```ruby
def success
	token_hash = UserCommunicationMethods::Email.decode_url_token(params.require(:base64_token))
	id = token_hash.delete(:id)

	@ucm = UserCommunicationMethod.includes(:user).find_by(id: id) if id.present?
	@patient = @ucm&.user
end
```

> [!Tip]
> La clave es que en `EmailVerificationsController` se redirige a una página donde se muestra un enlace al Clinical Dashboard del Provider. En el redirect de `PatientEmailVerificationsController` no hay tal enlace.


## Solución

Un controlador base que tenga toda la lógica general y controladores hijos que hagan las diferencias.

## Pruebas

Para más detalles ver:

- [[Probando Email Verification Landing]]
- [[Probando Salida de Correos en Local]]

Detalles para este caso a continuación.

