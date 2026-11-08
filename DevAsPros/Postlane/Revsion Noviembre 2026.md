# Revisión 7 de Noviembre de 2026

## Duración del token de sesión

Ahora mismo está así:
```ruby
exp: 7.days.from_now.to_i,
```

7 días. Creo que está bien.

## Payload del token

Ahora mismo solo es el user_id:
```ruby
JsonWebToken.encode(payload: { user_id: user.id })
```

¿Hay que cambiarlo a incluir más cosas?

## Respuesta del endpoint de sesión

Solo devuelve user y token. Hay que hacer que devuelva información de la suscripción.