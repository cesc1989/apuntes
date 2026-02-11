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

## Wrapper con clase `field_with_errors` rompe el form de registro

Solución poner esto:
```ruby
ActionView::Base.field_error_proc = Proc.new do |html_tag, instance|
  html_tag
end
```

en `config/initializers/field_error.rb`.

## Mostrar error del campo en el mismo lugar

En vez de mostrar una lista de errores como hace el partial de devise:
```html
<% if resource.errors.any? %>
  <div class="alert alert-danger">
    <h5>
      <%= I18n.t("errors.messages.not_saved", count: resource.errors.count, resource: resource.class.model_name.human.downcase) %>
    </h5>

    <ul>
      <% resource.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

Mejor mostrar cada error en el campo correspondiente. Se logra así:
```html
<div class="form-floating mb-3">
	<%= f.text_field :first_name,
			autofocus: true,
			class: "form-control #{'is-invalid' if resource.errors[:first_name].any?}",
			placeholder: "Nombre" %>
	<%= f.label :first_name, "Nombre" %>

	<% if resource.errors[:first_name].any? %>
		<div class="invalid-feedback d-block">
			<%= resource.errors[:first_name].first %>
		</div>
	<% end %>
</div>
```

Hay que agregar la clase "is-invalid" y boostrap hace el resto.