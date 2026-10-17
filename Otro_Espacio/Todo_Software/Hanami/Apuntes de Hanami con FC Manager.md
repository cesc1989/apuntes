# Apuntes mientras hago FC Manager junto con Deep Seek

La parte inicial del modelo de datos, migraciones y relaciones lo hice yo. Luego le cedí más control a Deep Seek para migrar la maquetación y estilos desde el prototipo al proyecto. Ha hecho varias cosas raras entonces aquí apunto lo que aprendo de todo eso.

Esto es lo que quiero revisar:

- [x] Contracts en create action
- [x] PlayerForm en edit action
- [x] handle(*, response) en new action
- [ ] upsert de ROM en PlayerRepo
- [x] Forms:
    - [x] se pasa values en el partial
    - [x] se pasa el method y el cancel paths
- [x] Views:
    - [x] layout = app
    - [x] exposures
- [ ] Helpers:
    - [ ] Hay bastante código. Revisar para tener claridad.
- [x] Lib:
    - [x] PlayersCatalog y TraitsCatalog


## Contracts::PlayerParams

Pasan varias cosas aquí. Esto es lo que generó DipSik en la ruta `lib/fcmanager/contracts/player_params.rb`:
```ruby
# frozen_string_literal: true

require "dry/validation"

module Fcmanager
  module Contracts
    # Validación de params para crear y actualizar un jugador.
    # `:id` es opcional para poder reutilizarlo en el update.
    class PlayerParams < Dry::Validation::Contract
      params do
        optional(:id).filled(:integer)

        required(:player).hash do
          required(:name).filled(:string)
          required(:age).filled(:integer, gteq?: 15, lteq?: 45)
          required(:position).filled(:string)
          required(:role).filled(:string)
          required(:overall).filled(:integer, gteq?: 1, lteq?: 99)
          required(:height).filled(:integer, gteq?: 150, lteq?: 210)
          required(:foot).filled(:string)
          optional(:comment).maybe(:string)

          # Traits dinámicos: se conservan como hash y se sanitan en la acción.
          optional(:traits).hash
        end
      end
    end
  end
end
```

Cosas que llaman mi atención:
- La ubicación del archivo
- Requerir `dry/validation`
- El uso de `optional`
- Los operadores de menor que y mayor que

Trataré de responder.

### La ubicación del Archivo

Creo que DipSik lo creó en esta ruta ya que esta clase se usa en las actions Create y Update. Otra razón (me dijo DipSik) es que ya había la carpeta `lib/fcmanager` así que le pareció razonable ubicar este tipo de componentes que no son web fuera de `app/`.

## Requerir dry/validation y usar Dry::Validation::Contract en vez de Hanami::Action::Params

En los docs (https://hanakai.org/learn/hanami/v3.0/actions/parameters#parameter-validation) muestran que la clase se puede heredar de `Hanami::Action::Params`, sin embargo, según DipSik, la clase Params no provee los métodos necesarios para la validación del Schema. En cambio `Dry::Validation::Contract` sí lo hace.

Docs de Dry Validation: https://hanakai.org/learn/dry/dry-validation/v1.11

### Terminología de Dry Validation

Esto es un esquema:
```ruby
params do
	optional(:id).filled(:integer)

	required(:player).hash do
		required(:name).filled(:string)
		required(:age).filled(:integer, gteq?: 15, lteq?: 45)
	end
end
```

#### Schema

Docs: https://hanakai.org/learn/dry/dry-schema/v1.14

El esquema válida la estructura de datos.

Casos de uso para validar:
- Form params
- “GET” params
- JSON documents
- YAML documents
- Application configuration (ie stored in ENV)
- Replacement for `strong-parameters`

#### Macros

Son las reglas para hacer la validación. Docs: https://hanakai.org/learn/dry/dry-schema/v1.14/basics/macros

Las que he visto en el uso de Hanami son:

- `hash`
- `value`
- `filled`
- `maybe`

La diferencia ente `value` y `filled` es que el primero sirve para generar una lista de expectativas. El segundo es para indicar que el valor no puede ser nulo.

Ejemplo de `value`:
```ruby
required(:age).value(:integer, gt?: 18)
```

Aquí se expresa que el valor debe ser entero y mayor a 18.

Ejemplo de `filled`:
```ruby
required(:role).filled(:string)
```

Se espera que el valor sea un string no vacío. No permite nulo.

#### Predicates

Los docs: https://hanakai.org/learn/dry/dry-schema/v1.14/basics/built-in-predicates

Los predicados son las opciones que usa el macro. Sirven para verificar la validez de cada entrada. En lo definido arriba se usan:

- `gteq?`
- `lteq?`

## Clase PlayerForm

Está en `lib/fcmanager/player_form.rb`. Es básicamente un espacio para poner código que ayuda a cargar/convertir valores en el form de Player.

En Hanami se usa el Relation para definir la conexión con la base de datos y el repo para definir las consultas. El código en está clase no tiene cabida en otro lugar. Se puede poner el action pero como se va a reusar supongo que DipSik tomó la decisión de agruparlo.

Por ejemplo, la función `attributes(params)` sirve para preparar los valores ingresados en el form para ser guardados en la BD.

La función `values(player)` hace lo contrario. Toma los valores de la base de datos y los prepara para el form.

## Sobre handle con y sin splat

En algunas partes de los docs lo ponen así:
```ruby
handle(*, response)
```

Y en otras así:
```ruby
handle(*, response)
```

¿por qué?

La explicación al final me la dio claudio:
> `*` es un splat *anónimo*: recoge cero o más argumentos posicionales en un array sin nombre, que no se puede usar dentro del método.
>
> Cuando hay parámetros obligatorios después de un splat, Ruby los llena primero desde el final, y el splat se queda con lo que sobre al principio. Por eso `response` siempre recibe el último argumento.

Esto porque había probado creando este método en el Laboratorio con varias ejecuciones:
```ruby
def handle(*, response)
  puts(response)
end

handle(response: "hola")
handle(request: "que", response: "mundo")
handle("que", "mundo")
```

que me dieron estas salidas:
```ruby
{response: "hola"}
{request: "que", response: "mundo"}
mundo
```

La explicación es que el parámetro `response` está recibiendo todo lo que le pase y por eso la salida es un hash en los llamados que son un hash (no son keyword arguments).

## PlayersCatalog

> [!Note]
> Esto me parece mucho trabajo pero al parecer, por lo que leo de Hanami, es lo que toca hacer. Hanami no da las cosas tan listas para solo usar como Rails.

Los Enums en Hanami no funcionan como en Rails. Así definí varios enums en la relation de Player:
```ruby
attribute(
	:role,
	Types::Integer,
	read: Types::String.enum(
		"promesa" => 0,
		"esporádico" => 1,
		"rotación" => 2,
		"importante" => 3,
		"esencial" => 4
	)
)
```

De esta forma se le dice que cuando se _lee_ el dato haga lo conversión. El tema es que aquí lo estoy usando mediante `read` types. Como estoy aprendiendo no me queda claro.

Lo que hizo DipSik en esta clase fue armar varios hashes que definen cómo se lee el valor de la base de datos, cómo se guarda, cómo se muestra en el form y una llave con la que se accede.

Para ello definió varias funciones según lo que se necesite.

Para cargar las opciones de los forms:
```ruby
module Fcmanager
  module PlayersCatalog
    module_function

    def position_options
      POSITIONS.map { |position| [position[:label], position[:value]] }
    end

    def role_options
      ROLES.map { |role| [role[:label], role[:value]] }
    end

    def foot_options
      FEET.map { |foot| [foot[:label], foot[:value]] }
    end
  end
end
```

Para guardar del form a la bd:
```ruby
module Fcmanager
  module PlayersCatalog
    module_function

    # Valor de formulario → entero para la DB.
    def position_db(value)
      POSITION_DB.fetch(value, 6)
    end

    def role_db(value)
      ROLE_DB.fetch(value, 1)
    end

    def foot_db(value)
      FOOT_DB.fetch(value, 0)
    end
	end
end
```

Y para leer de la BD y cargar en el form:
```ruby
module Fcmanager
  module PlayersCatalog
    module_function
    
		# Clave de la relación → valor de formulario (para precargar la edición).
    def position_value(key)
      POSITION_VALUE.fetch(key.to_s, "MC")
    end

    def role_value(key)
      ROLE_VALUE.fetch(key.to_s, "Esporadico")
    end

    def foot_value(key)
      FOOT_VALUE.fetch(key.to_s, "derecho")
    end
  end
end
```

## Views

### config.layout = app

Por alguna razón DipSik puso eso en todas las vistas. Se me hizo raro porque en el Getting Started no mencionan nada de eso. Reviso los docs y dice que por defecto es  `app`:

[Docs](https://hanakai.org/learn/hanami/v3.0/views/templates-and-partials)
> views use an `app` layout by default.


### Exposures

Este es el mecanismo en que la View le pasa valores que vienen del Action al Template. No es tan automático como en Rails.

El ejemplo más claro es pasar los `traits` hasta el form. Todo el viaje empieza en la action edit:
```ruby
module Fcmanager
  module Actions
    module Squad
      class Edit < Fcmanager::Action
        include Deps["repos.player_repo", "repos.trait_repo"]

        def handle(request, response)
          # (...)
          response[:traits] = trait_repo.all
        end
      end
    end
  end
end
```

> [!Important]
> Lo que está en `response[:traits]` es guardado en `response.exposures` y es pasado tal cual a la View.

Luego en la View tenemos acceso a ese valor:
```ruby
module Fcmanager
  module Views
    module Squad
      class Edit < Fcmanager::View
        include Deps["routes"]

        expose :traits
        # (...)
      end
    end
  end
end
```


En el template de edit no hay necesidad de pasarle el local `traits` al form. Eso se pasa de manera automática al partial:
```ruby
# template
<main>
  <section class="form-page">
    <div class="form-panel">
      <%= render "squad/form" %>
    </div>
  </section>
</main>

# partial
<h4 class="card-subtitle">Características</h4>
  <div class="traits">
    <% grouped_traits(traits).each do |group| %>
    <% end %>
  </div>
```

## Forms y el partial que pasa muchos locals

Esto hizo DipSik en los templates new y edit.

new:
```ruby
<%= render "squad/form",
            action: routes.path(:squad),
            form_method: :post,
            values: {
              name: "",
              age: 20,
              position: "MC",
              role: "Esporadico",
              overall: 65,
              height: 180,
              foot: "derecho",
              comment: ""
            },
            traits: traits,
            cancel_path: routes.path(:squad),
            submit_label: "Guardar" %>
```

Edit:
```ruby
<%= render "squad/form",
            action: routes.path(:update_squad, id: player_id),
            form_method: :patch,
            values: player_values,
            traits: traits,
            cancel_path: routes.path(:squad),
            submit_label: "Guardar cambios" %>
```

Hay cosas que sí toca pasar y otras que tal vez DipSik no usó bien.

### Lo que sí toca pasar

> [!Info]
> Claudio dice:
> > Hanami no tiene `form_with model:` ni rutas polimórficas, así que la URL, el método y el texto del botón sí hay que pasarlos.

Eso explica que haya que pasar:
- `action`
- `form_method`
- `submit_label`

Claudio explica por lo siguiente:
> No hay objeto modelo con `persisted?`, así que no puede deducir POST o PATCH, la URL ni "Crear" o "Guardar cambios". `action` y `form_method` hay que pasarlos sí o sí.


> Los errores no van pegados al modelo. Viven en `request.params.errors`, y de ahí salen los helpers `field_errors` y `error_messages`.

Por lo anterior es que tenemos esto en el partial del form. Justo al comienzo:
```ruby
<% errors = request.params.errors %>
```

### Lo que sobró

Sobraba el local `cancel_path`. Se está llamando a la misma ruta.

### Refactor con Claudio

Claudio notó que se podía refactorizar para no tener que pasar tantos locals. Quedó mucho mejor. Más "Hanami Way" usando exposures para pasar los locals automáticamente al template/partial.

Ver en este commit: https://github.com/cesc1989/fcmanager/commit/6be69c50dc98a045cb2bbed9c96f664c15352a14