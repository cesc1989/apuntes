# Apuntes Testing en Ruby y Rails. Parte 2

# RSpec mandando string vacíos en vez de nil

O enviando valores booleanos envueltos en string.

En vez de enviar:

```ruby
{
  boolean_field: true
}
```

Envía:

```ruby
{
  boolean_field: "true"
}
```

En esta prueba en provider portal

```ruby
post(
  "#{base_api_url}/plans_of_care",
  params: { data: { physician_ids: [] } }
)
```

Necesitaba mandar el array vacío. Sin embargo, RSpec decide enviarlo como un array con una cadena vacía:

    (byebug) ids
    [""]

Encuentro, [según este issue](https://github.com/rspec/rspec-rails/issues/2021), que es un error más de Rails que de RSpec.

> (…)this is how Rails behaves, you can't ever have a nil parameter passing through, `http://myapp/path/to/controller?query=` would come through as `{ "query" => "" }`
> 
> if you want nil as a value you need to omit the key e.g. `http://myapp/path/to/controller` which should make params `{}`
> 
> (…) because `GET "/users?query"` is invalid, `query` here isn't a param, its junk.

Acá encuentro [la forma de solucionarlo](https://stackoverflow.com/a/44704018/1407371).

```ruby
post(
  "#{base_api_url}/plans_of_care",
  params: { data: { physician_ids: [] } },
  as: :json
)
```

También sirve para que RSpec envíe el tipo de dato como es. Ejemplo:

```ruby
before do
  post(
	url,
	params: { 
	  form: {  type_name: 'odi', delete_existing_answers: false }
	}
  )
end
```

Enviará `delete_existing_answers` como `"``false``"` y no como un boolean. Para que llegue como boolean, debe ser así la petición:

```ruby
before do
  post(
	url,
	params: {
	  form: { type_name: 'odi', delete_existing_answers: false }
	},
	as: :json
  )
end
```
