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

# Personalizando los hooks de RSpec y documentación

Estaba probando como hacer una configuración que solo afectara los tests de `patient_self_report` que metí en Edge. Quería lograr la forma que lo que creara solo afectara a cierto grupo de carpetas.

Encontré [esto en Stack Overflow](https://stackoverflow.com/a/29907583/1407371). Darle un metadato a los tests según la carpeta y luego ejecutar los hooks buscando ese metadato:
```ruby
config.define_derived_metadata(file_path: Regexp.new("/spec/requests/patient_self_report/")) do |metadata|
  metadata[:patient_self_report] = true
end

config.before(:suite, :patient_self_report) do
end
```

Sin embargo, eso no me funcionó en Edge por la forma en que se corren los tests en el CI.

## Documentación de RSpec

La documentación de la página oficial es bastante sencilla y es más enfocada en el uso de la gema. Si se quiere saber sobre las configuraciones toca ir por otro lado. En el post de Stack Overflow enlazan a una página de documentación donde se puede ver lo de `derived_metadata` ([aquí](https://www.rubydoc.info/github/rspec/rspec-core/RSpec/Core/Configuration:define_derived_metadata)) y los [Hooks](https://www.rubydoc.info/github/rspec/rspec-core/RSpec/Core/Hooks).