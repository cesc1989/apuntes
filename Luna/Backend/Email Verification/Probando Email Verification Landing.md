# Probando las dos landings de Email Verification

Tanto success como failure. Aquí listo los pasos para generar un token bueno para poder probar esta característica.

## Generar token bueno y Verificación Ok

Necesitamos una instancia de `UserCommunicationMethod` de tipo Email. A esta le asignamos un código de verificación:
```ruby
ucm = UserCommunicationMethod.email.order("RANDOM()").first
ucm.update!(verification_code: SecureRandom.urlsafe_base64(16))
```

Para poder abrir este código desde una URL hay que hacer un encodeo:
```ruby
Rails.application.routes.url_helpers.verify_email_url(
	UserCommunicationMethods::Email.encode_url_token(ucm),
	host: ENV.fetch("ROOT_URL"),
	protocol: Luna.env_protocol,
	port: ENV["APPLICATION_PORT"]
)

# => "http://localhost:3000/verify_email/eyJpZCI6IjAwMDA3MTQ5LTViNTAtNGQ0Mi05MDBlLTFlYmFjYTQxZDA1NiIsImNvZGUiOiJMQzJVMnR5OTdLQmt0Uk9RUUxPTjZBIn0="
```

Con esa URL ya se puede probar abrir enlace verificable o no.

## Token malo y Fallo en verificación

Para que la verificación no se dé hay que usar un UCM que:

- `verification_code` no coincida con el código que se extraiga de la URL
- o que `expired_verification_code?` de true

Así es como se determina si el código expiró:
```ruby
def expired_verification_code?
  verification_code.present? &&
    verification_code_expires_at.present? &&
	verification_code_expires_at < Time.current
end
```

Entonces:

```ruby
bad_ucm = UserCommunicationMethod.email.order("RANDOM()").first
bad_ucm.update!(
  verification_code: "holahola",
  verification_code_expires_at: 5.days.ago
)
```

Y generamos una URL para ese UCM malo:
```ruby
Rails.application.routes.url_helpers.verify_email_url(
	UserCommunicationMethods::Email.encode_url_token(bad_ucm),
	host: ENV.fetch("ROOT_URL"),
	protocol: Luna.env_protocol,
	port: ENV["APPLICATION_PORT"]
)

# => "http://localhost:3000/verify_email/eyJpZCI6ImRkNTRiNDYzLWU2YTYtNGYxNC04ODVlLTU1MzVmY2M4YjVkMCIsImNvZGUiOiJ0aTU5a3JlVzdsYVpuelRGbHZJVTRnIn0="
```

# Pruebas en Alfa

Se necesita saber que el correo de verificación llegue a un correo que podamos abrir.

Entonces podemos intentar buscar un UCM que tenga un correo conocido:
```ruby
ucm = UserCommunicationMethod.email.where(value: "francisco.quintero@ideaware.co").first
```

```ruby
ucm = UserCommunicationMethod.where("value LIKE '%francisco.quintero%'")

UserCommunicationMethod.find_by(value: "francisco.quintero+lunaindianapolis@ideaware.co").update(verification_status: "unverified")

ucm = UserCommunicationMethod.find_by(value: "francisco.quintero+lunaindianapolis@ideaware.co")
```

Ya con este podemos probar hacer una URL verificable:
```ruby
Rails.application.routes.url_helpers.verify_email_url(
	UserCommunicationMethods::Email.encode_url_token(ucm),
	host: ENV.fetch("ROOT_URL"),
	protocol: Luna.env_protocol,
	port: ENV["APPLICATION_PORT"]
)
```

# Query en Grafana para ver si el worker se movió

```
{app="backend-sidekiq-worker"} |= `EmailVerificationWorker`
```