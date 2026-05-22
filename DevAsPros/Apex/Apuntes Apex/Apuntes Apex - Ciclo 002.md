# Apuntes Apex - Ciclo 002

# Mejoras a scripts de despliegue

Quiero fortalecer los scripts de despliegue. He visto muchos casos donde completan con éxito pero en los logs hubo fallos.

## Usando pipefail en scripts de despliegue

DeepSeek recomendó esto:
```bash
#!/bin/bash

set -euo pipefail
```

En cada script de despliegue. Pero después de varios problemas tocó solo dejarlo en:
```bash
set -eo pipefail
```

### Error de chruby `PREFIX: unbound variable` 🟢

> [!Note]
> Hay problema de chruby con `set -u`
>
> Ver: https://github.com/postmodern/chruby/issues/417

Al poner esa línea en el script `003_after_deploy.sh` da este error en github actions:
```
/usr/local/share/chruby/chruby.sh: line 4: PREFIX: unbound variable
```

> [!Important]
> La solución fue no usar `-u` porque chruby no juega bien con eso.

Se han probado varias cosas pero nada parece resultar. Probé todo esto.

**Exportar PREFIX**
```bash
export PREFIX="${PREFIX:-/usr/local}"

. /home/ubuntu/.profile
```

**Desactivar -u y reactivarlo**
```bash
set +u
. /home/ubuntu/.profile
set -u
```

Combinar las anteriores
```bash
set +u
export PREFIX="${PREFIX:-/usr/local}"
. /home/ubuntu/.profile
set -u
```

Set después de cargar el `.profile`
```bash
. /home/ubuntu/.profile

set -eo pipefail
```

### Detalle Importante: scripts no llegaban al servidor 🔑

> [!Danger]
> El problema es que todos los cambios que mandé nunca fueron ejecutados porque el servidor se quedó con los archivos donde el despliegue fallaba.
> 
> Cada vez que se corría el action del despliegue, se estaban ejecutando los scripts dañados.

Si agrego este paso al despliegue puedo copiar los scripts antes de ejecutarlos:
```yml
- name: Upload scripts to VPS
	run: |
		scp -r scripts/* supermenu_cloud:~/supermenu/deployments/api-release/scripts/  
```

> [!Warning]
> Esto tiene un problema. Es que modifica los archivos y causa que el pull falle porque "hay cambios" en el repo git que tiene el servidor.


# Usar Nested Form con Stimulus Component

Etiquetas: #supermenu

Este es el componente: https://www.stimulus-components.com/docs/stimulus-rails-nested-form/

Las instrucciones de uso son sencillas y fácil de seguir. Todo funcionó normal excepto un detalle.

## Quitar elemento existente del nested form

Cuando se agrega un nuevo elemento y se clica el botón de quitar, este se quita de la vista.

Sin embargo, cuando se clica el botón quitar de un elemento existente (cargado de la BD), no se quita. Esto pasa porque la funcionalidad del componente (`RailsNestedForm#remove`) trabaja de dos formas diferentes.

Cuando el elemento es nuevo, busca leer el data attribute `newRecord`. Cuando no, tira a ocultar el elemento con `display: none`:
```js
if (wrapper.dataset.newRecord === "true") {
      wrapper.remove()
    } else {
      wrapper.style.display = "none"
```

[Fuente](https://github.com/stimulus-components/stimulus-components/blob/master/components/rails-nested-form/src/index.ts#L32C5-L35C37)

En el caso de Super Menú así se define el elemento:
```html
<div class="nested-form-wrapper variant-row d-flex gap-2 mb-2 align-items-end" data-new-record="<%= vf.object.new_record? %>">
 # ...
</div>
```

En el caso de Super Menú esto no funciona porque se usa la clase de Boostrap `d-flex`. Esta clase pone un `!important`:
```css
.d-flex {
  display: flex !important;
}
```

Dicho important invalida la clase inline que agrega RailsNestedForm así que no se oculta.

La solución es extender el componente con un controlador Stimulus del proyecto y modificar el método `remove` para que haga algo con respecto a los elementos existentes.

### Extender RailsNestedForm

Al final así lo resolví, con la ayuda de DeepSeek:
```js
import RailsNestedForm from "@stimulus-components/rails-nested-form"

export default class extends RailsNestedForm {
  remove(e) {
    e.preventDefault()

    const wrapper = e.target.closest(this.wrapperSelectorValue)

    if (wrapper.dataset.newRecord === "true") {
      wrapper.remove()
    } else {
      /* Esto es lo diferente al componente original */
      wrapper.classList.add("d-none")

      const input = wrapper.querySelector("input[name*='_destroy']")

      if (input) input.value = "1"
    }
  }
}
```

Funciona todo al pelo. Agregar y borrar elementos del formulario.

# Todos los pasos para agregar confirmación de correo en Super Menu 📧

Estos son todos los commits de cuando se hizo en Enlacito.

## Campos en la BD 🟢

Commit: https://github.com/cesc1989/enlacito/commit/709e5200f0b62a93eceeccf9e20355b331aaa303

Migración de campos de Devise:
```ruby
class AddConfirmableToUsers < ActiveRecord::Migration[7.1]
  def up
    add_column :users, :confirmation_token, :string
    add_column :users, :confirmed_at, :datetime
    add_column :users, :confirmation_sent_at, :datetime

    # Este campo va de la mano de la opción reconfirmable del initializer
    add_column :users, :unconfirmed_email, :string

    add_index :users, :confirmation_token, unique: true

    User.update_all confirmed_at: DateTime.now
  end

  def down
    remove_index :users, :confirmation_token
    remove_columns(
      :users,
      :confirmation_token,
      :confirmed_at,
      :confirmation_sent_at
    )
  end
end
```

En el modelo User se agrega la línea `:confirmable` al macro `devise`.

## Configuración de Resend en el repo 🟢

Commit: https://github.com/cesc1989/enlacito/commit/09810c855a1def352b34bba03a986bfdcab7b047

Agregar gema de resend:
```ruby
gem "resend", "1.0.1"
```

Agregar initializer en `config/initializers/resend.rb`
```ruby
Resend.api_key = ENV.fetch("SUPERMENU_RESEND_API_KEY", "")
```

Y archivo `.env.test` para las pruebas:
```bash
SUPERMENU_RESEND_API_KEY="loquesea"
```

## Actualización de ActionMailer en environments 🟢

Commit: https://github.com/cesc1989/enlacito/commit/8fc7dac6006ad9709d8a850e63a84deab25361e4

En `config/environments/development.rb`:
```ruby
config.action_mailer.default_url_options = { host: "localhost", port: 3006 }
```

En `config/environments/production.rb`:
```ruby
config.action_mailer.default_url_options = { host: "supermenu.devaspros.com" }
config.action_mailer.delivery_method = :resend
```

En `config/environments/test.rb`:
```ruby
config.action_mailer.default_url_options = { host: "localhost:3000" }
```

## Preparación de los factories 🟢

Commit: https://github.com/cesc1989/enlacito/commit/ef1854fac4a94e59a4f301bdd334defe33044a15

En `spec/factories/users.rb`:
```ruby
trait :confirmed do
	confirmed_at { DateTime.current }
end
```

## Configuración de Devise 🟢

Commit: https://github.com/cesc1989/enlacito/commit/d906b5ab275a79bab8a5beab238bf3705ab0c3a2

Se cambia en `app/controllers/users/registrations_controller.rb` el método para que después de completar el registro se devuelva al login.
```ruby
  # The path used after sign up for inactive accounts.
  def after_inactive_sign_up_path_for(resource)
    # super(resource)
    new_user_session_path
  end
```

Verificar que la vista tenga lo necesario. Vista en `app/views/devise/mailer/confirmation_instructions.html.erb`.

Y se configura para que el token de confirmación venza en 24 horas en el initializer:
```ruby
config.confirm_within = 24.hours
```

También usar el dominio verificado en Resend:
```ruby
config.mailer_sender = "noreply@resend.supermenu.devaspros.com"
```

## Cargar todas las vistas de Devise 🟢

Para no joder más con eso. Commit: https://github.com/cesc1989/enlacito/commit/a555bc57e92b95350c5c466abcfbb0701fe819ce

## Usar el subdominio verificado en los mailers 🟢

Commit: https://github.com/cesc1989/enlacito/commit/efe4033b9c619f3d165dc31adf120e22faed2054

En `app/mailers/application_mailer.rb`:
```ruby
default from: "noreply@resend.supermenu.devaspros.com"
```

## Adecuar Mailer y Vistas 🟢

Commit: https://github.com/cesc1989/enlacito/commit/37637121b2193cd1772b7f6d03930c4c73348aa6

Crear nuevo mailer que herede del devise para poder modificar valores.
```ruby
# app/mailers/super_menu_mailer.rb
class SuperMenuMailer < Devise::Mailer
  layout "mailer"
  default from: "noreply@resend.supermenu.devaspros.com"
end
```

Devise debe usar este nuevo mailer:
```ruby
# config/initializers/devise.rb

config.mailer = "SuperMenuMailer"
```

En `config/environments/development.rb` configura las previews del mailer:
```ruby
config.action_mailer.preview_paths << Rails.root.join("spec/mailers/previews")
```

Crea las previews en `spec/mailers/previews/devise_mailer_preview.rb`:
```ruby
class DeviseMailerPreview < ActionMailer::Preview
  def confirmation_instructions
    user = User.new(email: "usuario@ejemplo.com")
    SuperMenuMailer.confirmation_instructions(user, "fake_token_para_preview")
  end

  def reset_password_instructions
    user = User.new(email: "usuario@ejemplo.com")
    SuperMenuMailer.reset_password_instructions(user, "fake_token_para_preview")
  end
end
```
