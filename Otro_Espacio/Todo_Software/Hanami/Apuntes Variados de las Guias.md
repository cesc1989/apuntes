# Apuntes Variados de las Guías Oficiales

## Cómo insertar nuevos registros de una Relation

```ruby
books_relation = app["relations.books"]
books_relation.insert(title: "Test Driven Development", author: "Kent Beck")
```

## Sobre los assets

El build es con [esbuild](https://esbuild.github.io/).

Comandos:
- `hanami assets compile`: los prepara para producción
- `hanami assets watch`: para en desarrollo compilar al cambiarlos

### Los Entrypoints

El compilado depende de los entrypoints:
> When Hanami compiles your assets, it detects your JavaScript **entry points**

Definición:
> An **entry point** is a file that serves as the starting point for a compiled **asset bundle**

El entrypoint por defecto es `app/assets/js/app.js`. Viene así al arrancar una app nueva:
```js
import "../css/app.css";
```

> [!Importante]
> La guía dice:
> Solo los archivos JS y CSS referenciados en un entrypoint serán incluídos en el bundle luego de compilar.


#### ¿Hay que incluir siempre el CSS en el entrypoint? 🟡🚨

Respuesta:???

#### Múltiples Entrypoints

La guía dice:
> You can have as many entry points as you like. You might create dedicated entry points for certain pages or features in your app, to improve page loading and rendering.

Así se pueden tener entrypoints para login, landing, admin, etc.


#### Crear un nuevo entrypoint

Se crean dentro de la carpeta `assets/js`. Se crea una nueva carpeta y luego el archivo debe ser `app.js`.

Ejemplo para login: `app/assets/js/login/app.js`. Y lo que podría haber:
```js
import "../../css/login/app.css";
import { resetPassword } from "./resetPassword";
```

Ejemplo de la app con dos entrypoints. El por defecto y el de login:
```bash
app/assets
├── css
│   ├── app.css
│   └── login
│       └── app.css
├── images
│   └── favicon.ico
└── js
    ├── app.js # Entry point
    └── login
        ├── app.js # Entry point
        └── resetPassword.js
```


### Usar los assets

Se pueden acceder en las _views_ o mediante el _assets component_.

El _assets component_ es un objeto registrado en `"assets"` la app (o el Slice).

En una consola de hanami:
```
app["assets"]
#<Hanami::Assets:0x0000000124856928
 @config=#<Hanami::Config::Assets:0x0000000123cb0f60 @__config__=#<Dry::Configurable::Config values={serve: true}>, @base_config=#<Hanami::Assets::Config:0x0000000123c7bd38 @__config__=#<Dry::Configurable::Config values={node_command: "node", path_prefix: "/assets", subresource_integrity: [], base_url: #<Hanami::Assets::BaseUrl:0x0000000124851e28 @url="">}>>>,
 @root=#<Pathname:/Users/francisco/projects/hanami-test/bookshelf/public/assets>>

Hanami.app["assets"]
#<Hanami::Assets:0x0000000124856928
 @config=#<Hanami::Config::Assets:0x0000000123cb0f60 @__config__=#<Dry::Configurable::Config values={serve: true}>, @base_config=#<Hanami::Assets::Config:0x0000000123c7bd38 @__config__=#<Dry::Configurable::Config values={node_command: "node", path_prefix: "/assets", subresource_integrity: [], base_url: #<Hanami::Assets::BaseUrl:0x0000000124851e28 @url="">}>>>,
 @root=#<Pathname:/Users/francisco/projects/hanami-test/bookshelf/public/assets>>

app["assets"]["app.js"]
#<Hanami::Assets::Asset:0x0000000123efdef8 @base_url=#<Hanami::Assets::BaseUrl:0x0000000124851e28 @url="">, @path="/assets/app.js", @sri=nil>
```


#### Helpers

También tiene los similares que hay en Rails:

- `asset_url`
- `javascript_tag`
- `stylesheet_tag`
- `image_tag`
- `favicon_tag`
- `video_tag`
- `audio_tag`


## Sobre Containers y Componentes 🟡🚨

[Docs](https://hanakai.org/learn/hanami/v3.0/app/container-and-components)

El código que se agrega a la carpeta `app/` se organiza automaticamente en un Container. Los Containers son la unidad base del Component Management System que usa Hanami.

Un Componente es un objeto que se crea para completar tareas dentro del sistema.

> [!Importante]
> Es clave que cada componente solo haga una tarea. Se mezclan componentes mediante Inyección de Dependencias (_Dependency Injection_)

> [!Note]
> > Hanami is designed to make it easy to create applications that are systems of well-formed components with clear dependencies.

Dan este ejemplo de un objeto que se vuelve componente:
```ruby
# app/slugifier.rb

module Bookshelf
  class Slugifier
    def call(title)
      title.downcase.gsub(/\s+/, "-")
    end
  end
end
```

La instancia de `Bookshelf::Slugifier` estará disponible como la key `"slugifier"` en `app` Container:
```ruby
bookshelf[development]> Hanami.app["slugifier"]
=> #<Bookshelf::Slugifier:0x000000010577afc8>
```

> [!Warning]
> En development, hay que correr `Bookshelf::App.boot` para que se puedan cargar todos los componentes en el contenedor `app`. Lo dice la misma consola:
> > Warning: Bookshelf::App is not booted. Run `Bookshelf::App.boot` to load all components, or launch the console with `--boot`.

### Dependency Injection en Hanami

Dan este ejemplo de como lograrlo en PORO:
```ruby
# app/books/create.rb

module Bookshelf
  module Books
    class Create
      attr_reader :book_repo
      attr_reader :slugifier

      def initialize(book_repo:, slugifier:)
        @book_repo = book_repo
        @slugifier = slugifier
      end

      def call(title:, author:)
        book_repo.create(
          title: title,
          author: author,
          slug: slugifier.call(title)
        )
      end
    end
  end
end
```

Se le pasa `book_repo` y `slugifier` al componente `books.create` así no necesita saber cómo instanciar ni qué funciones usar.

Hanami mejora esto mediante el mixin `Deps`:
```ruby
# app/books/create.rb

module Bookshelf
  module Books
    class Create
      include Deps[
        "repos.book_repo",
        "slugifier"
      ]

      def call(title:, author:)
        book_repo.create(
          title: title,
          author: author,
          slug: slugifier.call(title)
        )
      end
    end
  end
end
```

### Mixin Deps

Se usa incluyendolo en la clase: `include Deps["key"]`. Esto nos ahorra el setter/getter y la inicialización.

La `key` que recibe es la `key` con la que el componente se hace disponible en el contenedor (en este caso `app`).

Ejemplo, así se ven los componentes del Bookshelf de aprendizaje de las guías:
```ruby
Bookshelf::App.boot
app.keys

[# (... omito los por defecto)
 "repos.book_repo",
 "actions.books.create",
 "actions.books.destroy",
 "actions.books.edit",
 "actions.books.index",
 "actions.books.new",
 "actions.books.show",
 "actions.books.update",
 "actions.home.index",
 "views.books.edit",
 "views.books.index",
 "views.books.new",
 "views.books.show",
 "views.home.index"]
```

Y así fue como en la View usé el componente del repositorio:
```ruby
# frozen_string_literal: true

module Bookshelf
  module Views
    module Books
      class Edit < Bookshelf::View
        include Deps["repos.book_repo"]

        expose :book do |id:|
          book_repo.get(id)
        end
      end
    end
  end
end
```

## Migraciones

### Llaves Primarias 🔑

Se puede crear una llave primaria normalmente así:
```ruby
create_table :clubes do
	primary_key :id
end
```

En el FC Manager, cuando iba a crear `player_traits` me di cuenta en la [documentación de Sequel](https://sequel.jeremyevans.net/rdoc/files/doc/schema_modification_rdoc.html#label-create_join_table) que podía definir la llave primaria así:
```ruby
ROM::SQL.migration do
  change do
    create_table :player_traits do
      foreign_key :player_id, :players
      foreign_key :trait_id, :traits

      primary_key [:player_id, :trait_id] # <= esto
    end
  end
end
```

**Una llave compuesta.**

Según DeepSeek, esto se puede porque Sequel lo permite. En cambio en Rails no se puede eso nativamente. Por eso siempre vi todo con `:id` en Rails.

> [!Note]
> Sin embargo, Deep Seek explica que, para el caso de esta tabla, tiene más sentido usar `:id` como llave primaria porque no es solamente una _join table_ sino una entidad (porque está la columna `value`).

Finalmente, DeepSeek sugirió más bien agregar una [restricción única](https://sequel.jeremyevans.net/rdoc/files/doc/schema_modification_rdoc.html#label-unique) con la combinación de las columnas para prevenir crear el mismo trait más de una vez para el mismo jugador. Quedando la migración así:
```ruby
ROM::SQL.migration do
  change do
    create_table :player_traits do
      primary_key :id

      foreign_key :player_id, :players
      foreign_key :trait_id, :traits

      column :value, :integer, default: 0, null: false

      unique [:player_id, :trait_id]
    end
  end
end
```

## Relaciones

### Enums

Así se define un enum:
```ruby
module Fcmanager
  module Relations
    class Traits < Fcmanager::DB::Relation
      schema :traits, infer: true do
        attribute(
          :kind,
          Types::Integer,
          read: Types::String.enum(
            "fisicos" => 0,
            "mentales" => 1,
            "tecnicos" => 2,
            "arqueria" => 3
          )
        )
      end
    end
  end
end
```

La clave está en que se puede [definir tipos de salida y entrada](https://hanakai.org/learn/rom/v5.0/core-concepts/schemas#using-read-types) para forzar la conversión del dato.

En este caso queremos que la salida al ver un registro el valor que numérico pase a ser la representación en el enum. Así se ve en la consola:
```ruby
pp["relations.traits"].one
[fcmanager] [DEBUG] [2026-09-30 20:55:20 -0500] SQL sqlite 1ms SELECT `traits`.`kind`, `traits`.`id`, `traits`.`name` FROM `traits` ORDER BY `traits`.`id`
=> {kind: "fisicos", id: 1, name: "Aceleración"}
```

### Asociaciones

Esto es nada como en Rails. En Hanami (ROM) solo se trae la data que se pide.

Así definí las asociaciones para `clubs->players`:
```ruby
module Fcmanager
  module Relations
    class Players < Fcmanager::DB::Relation
      schema :players, infer: true do
        associations do
          belongs_to :club
        end
      end
    end
  end
end

module Fcmanager
  module Relations
    class Clubs < Fcmanager::DB::Relation
      schema :clubs, infer: true do
        associations do
          has_many :players
        end
      end
    end
  end
end
```

##### Combines

Ver [docs](https://hanakai.org/learn/rom/v5.0/core-concepts/combines)

> [!Important]
> ROM will **never** load associated data unless it is explicitly told to do so.

Hasta aquí nada raro. Lo raro es cuando voy a probar en la consola de Hanami. Para poder traer el Club de un Player (como haría en Rails) toca usar `combine` en la relación.

```ruby
players = app["relations.players"]
player = players.combine(:club).by_pk(1).one

{role: "rotación",
 dominant_foot: "izquierdo",
 primary_position: "cam",
 id: 1,
 name: "Yeims Rodriguez",
 age: 35,
 overall: 67,
 comment: nil,
 height: "1.40 cms",
 club_id: 1,
 secondary_positions: "cm",
 club: {id: 1, name: "Atletico Naciopan"}}
```

Y para acceder al club toca la sintaxis de hash:
```ruby
player[:club]

{id: 1, name: "Atletico Naciopan"}
```