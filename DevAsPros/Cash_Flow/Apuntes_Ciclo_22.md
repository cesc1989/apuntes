# Apuntes Ciclo 22

# Form edit con query params

Para poder cargar la lista de categorías en el formulario de nuevo gasto paso un query parameter al entrar al form:
```ruby
<%= link_to new_expenditure_path(parent_id: cat.id),
```

Y en el controlador uso el parámetro:
```ruby
def load_form_values
  @categories = Category.subcategories.where(parent_id: params[:parent_id])
end
```

Esto funciona bien al crear y editar con éxito. Pero cuando falla el form, al recargar la página la lista de categorías queda vacía.

¿Cómo le puedo pasar query params a la url del form?

## Crear nuevo recurso

Así se crea el form cuando es un nuevo recurso:
```ruby
<form data-turbo="false" action="/expenditures" accept-charset="UTF-8" method="post">

<input type="hidden" name="authenticity_token" value="CBN1Xm48c1FuxTTC1SiWoXsvmFLxsns4PY1Y4BRWfbBsN4IjDyJXBiqnH-r1mwY34svdvbuLPwS8QewhE8Q-Mg" autocomplete="off">
```

Atención con el atributo `action`:
```ruby
action="/expenditures"
```

y tiene método `post`. Esa ruta está definida como `POST /expenditures`.

## Actualizar recurso

En cambio, cuando se actualiza la ruta del formulario queda así:
```ruby
<form data-turbo="false" action="/expenditures/217" accept-charset="UTF-8" method="post">

<input type="hidden" name="_method" value="patch" autocomplete="off">

<input type="hidden" name="authenticity_token" value="EfC5KxDr2Qvqhj43cep7uCGinDlXZG6iqOUQMzIpdqk6kzYR7o3i3p74j7oVfQlVp1hFmUL6RokXAUHF36YdKA" autocomplete="off">
```

La ruta en `action` es `/expenditures/217` y usa un campo oculto para poder pasar el método `patch`.

## Enlaces sobre form_with

- form_with https://apidock.com/rails/ActionView/Helpers/FormHelper/form_with
- guías https://guides.rubyonrails.org/v7.0/form_helpers.html#dealing-with-model-objects
- Tuto en Medium https://medium.com/@eelan.tung/rails-forms-384cd22c65cc
- Otro tuto https://human-se.github.io/rails-demos-n-deets-2020/demo-resource-update/

# Campo `datetime_local_field` no aceptaba el input en Firefox Mobile

Desde que empecé a usar el campo `datetime_local_field` en el form, desde Firefox Mobile no se podía guardar gastos a menos que lo dejara tal cual y como cargaba el form.

Cuando carga el form se mostraba la fecha y hora con `DateTime.current`.

Probando con Ngrok, me di cuenta que la petición ni siquiera llegaba al servidor. Simplemente el botón guardar no hacía nada. El formulario tampoco mostraba errores y Firefox no indicaba nada.

Le pregunté a ChatGPT y la sugerencia fue forzar el formato esperado por el campo `datetime_local_field`. Lo hice así:
```ruby
<% expenditure_date = @expenditure.new_record? ? DateTime.current : @expenditure.spent_at %>

<%= form.datetime_local_field :spent_at, class: 'form-control', value: expenditure_date.strftime('%Y-%m-%dT%H:%M') %>
```

De esa forma pude probar y funcionaba de nuevo el form en Firefox Mobile. Todo era cuestión de forzar el formato. Según gepeto, es porque en navegadores Mobiles los estándares se aplican de manera más estricta.

# Asset  was not declared to be precompiled in production

Ya me había pasado antes este error en el Ciclo 17 -> [[Apuntes_Ciclo_17_-_Sidekiq_-_Cash_Flow#Problema de Assets Precompile con Capybara y System tests]]

```bash
Failure/Error: <%= javascript_importmap_tags %>

     ActionView::Template::Error:
       Asset `controllers/toast_controller.js` was not declared to be precompiled in production.
       Declare links to your assets in `app/assets/config/manifest.js`.

         //= link controllers/toast_controller.js

       and restart your server

   # --- Caused by: ---
	 # Sprockets::Rails::Helper::AssetNotPrecompiledError:
	 #   Asset `controllers/toast_controller.js` was not declared to be precompiled in production.
	 #   Declare links to your assets in `app/assets/config/manifest.js`.
	 #
	 #     //= link controllers/toast_controller.js
	 #
	 #   and restart your server
	 #   /Users/francisco/.gem/ruby/3.2.5/gems/sprockets-rails-3.4.2/lib/sprockets/rails/helper.rb:372:in `raise_unless_precompiled_asset'
```

Encontré una solución temporal en [este issue de 2020](https://github.com/rails/sprockets-rails/issues/458#issuecomment-618964357). Correr el comando:
```
bundle exec rake tmp:clear
```

Pero el error parece no haber tenido solución evidente.