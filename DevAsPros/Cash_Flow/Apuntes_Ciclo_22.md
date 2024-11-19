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