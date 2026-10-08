# Apuntes mientras hago FC Manager junto con Deep Seek

La parte inicial del modelo de datos, migraciones y relaciones lo hice yo. Luego le cedí más control a Deep Seek para migrar la maquetación y estilos desde el prototipo al proyecto. Ha hecho varias cosas raras entonces aquí apunto lo que aprendo de todo eso.

Esto es lo que quiero revisar:

- [x] Contracts en create action
- [ ] PlayerForm en edit action
- [ ] handle(*, response) en new action
- [ ] upsert de ROM en PlayerRepo
- [ ] Forms:
    - [ ] se pasa values en el partial
    - [ ] se pasa el method y el cancel paths
- [ ] Views:
    - [ ] layout = app
    - [ ] expose sin usar repos     
- [ ] Helpers:
    - [ ] Hay bastante codigo. Revisar para tener claridad.
- [ ] Lib:
    - [ ] PlayerCatalog y TraitsCatalog


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

