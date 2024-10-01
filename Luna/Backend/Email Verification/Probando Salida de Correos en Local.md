# Probar Salida de Correos en Local

En este documento listo el proceso para probar la salida de correos en Edge. Así no tener que esperar a que salga en alfa.

## Mailcatcher

Instala mailcatcher:

```
gem install mailcatcher --no-document
```

Si falla, ver [[Apuntes Rails on Rails - Parte 1#Error al intentar instalar mailcatcher]]

## Usando Mailcatcher

Corre el servidor con:
```
mailcatcher

Starting MailCatcher v0.10.0
==> smtp://127.0.0.1:1025
==> http://127.0.0.1:1080
*** MailCatcher runs as a daemon by default. Go to the web interface to quit.
```

Y accede en `http://127.0.0.1:1080`.

## Probar Envío de Solicitud de Verificación de Correo

Hay que encontrar un UserCommunicationMethod que se pueda verificar:
```ruby
bad_ucm = UserCommunicationMethod.email.order("RANDOM()").where(verification_status: "unverified").first bad_ucm.update!( verification_code: "holahola", verification_code_expires_at: 5.days.ago )
```

Luego precargarmos el Mailer:
```ruby
mailer = UserCommunicationMethods::EmailVerificationMailer.with(communication_method: bad_ucm, duration_hours: 12)
```

Finalmente, hacemos el envío:
```ruby
mailer.verification_email.deliver_now
```

Recargamos la página en Mailcatcher y debería salir el correo.

## Probar Weekly Reminder

```ruby
UserCommunicationMethods::WeeklyEmailVerificationReminderWorker.perform_in(5)
```