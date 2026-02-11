# Apuntes Ciclo 11 - Registro de Usuarios

## Permitir parámetros adicionales en el controlador de Devise

La tabla users tiene los campos `first_name` y `last_name` además de email y password. Para completar el registro en este caso toca agregarlos a la whitelist del controlador.

Hay que generar el controlador para poder modificarlo:
```bash
bundle exec rails generate devise:controllers users -c registrations
```

Y luego en la función `configure_sign_up_params` hay que modificar lo que va en `keys` así:
```ruby
keys: [:first_name, :last_name]
```

## Cambiar el mínimo de largo de la contraseña

Se puede hacer global en el inicializador `config/initializers/devise.rb`
```ruby
config.password_length = 10..128
```

O en el modelo:
```ruby
devise(
  :database_authenticatable,
  :validatable,
  password_length: 10..128
)
```

Visto en la wiki: https://github.com/heartcombo/devise/wiki/Customize-minimum-password-length