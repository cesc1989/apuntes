# Comanditos para Clinical Dashboard

Para crear cosas.

# Nuevo AdminUser

Desde una consola de Rails:
```ruby
password = SecureRandom.alphanumeric(12)
User.create(
  first_name: "Brandon",
  last_name: "Hite",
  email: "bhite@getluna.com",
  password: password
)
```

# Otros Comanditos

El de Patient Self Report -> [[Comanditos para Patient Self Report]]

El de Therapist Signup -> [[Comanditos para Therapist Signup]]