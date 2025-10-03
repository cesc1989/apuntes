# Primeros pasos en Phoenix

## Instalando Phoenix de manera local

Se instala con este comando:

```bash
mix archive.install hex phx_new
```

> install the Phoenix application generator

## Generar proyecto web con Phoenix

Siguiendo las instrucciones del libro, se creó la aplicación Phoenix de esta forma:
```
mix phx.new.web auction_web --no-ecto
```

Se hizo así dentro de la carpeta `cd auction_umbrella/apps/` y se especificó la bandera `--no-ecto` porque no tendrá conexión directa con la BD sino que la conexión será con el proyecto Auction.

Así se ve la carpeta `auction_umbrella` para mejor contexto:

```bash
## auction_umbrella

apps
├── auction
└── auction_web
```

## Otros Comandos

`mix phx.server` para lanzar el servidor.

`mix phx.routes` para listar las rutas.

`mix hex.info phoenix` para mostrar la versión de una librería.

# Ecto

Todo lo que hace Ecto

![[ecto.elixir.library.png]]

## Problema del Capítulo 7

[Docs](https://hexdocs.pm/phoenix/ecto.html)

En el capítulo 7 el autor menciona un archivo que nunca se creó. Lo copió del repo que se tiene  de para copiar código y configuro Ecto como se espera pero cuando corro la instrucción `mix ecto.create` da este error:

```
 mix ecto.create
warning: could not find Ecto repos in any of the apps: [:auction].

You can avoid this warning by passing the -r flag or by setting the
repositories managed by those applications in your config/config.exs:

    config :auction, ecto_repos: [...]
```

Esto mismo menciona otra persona en [un issue en ese repo](https://github.com/PhoenixInAction/phoenix-in-action/issues/4). Sin respuesta a día de hoy.

Solución...

En aplicaciones Umbrella, o se tienen todas las configs en el archivo central o es por cada proyecto. Cuando se tiene por cada proyecto, hay que importar sus configs en el archivo central en todo caso.

```ruby
# umbrella/config/config.exs

import Confit

# (...)

import_config "../apps/auction/config/config.exs"

# umbrella/apps/auction/config/config.exs

import Config

config :auction, Auction.Repo,
  username: "francisco",
  password: "",
  database: "auction_prueba",
  hostname: "localhost"

config :auction, ecto_repos: [Auction.Repo]

IO.inspect("olaa")
```

Y ahora sí funciona:
```bash
$ mix ecto.create -r Auction.Repo
"olaa"
Generated auction app
The database for Auction.Repo has already been created
```

## Crear la BD

Para crear la bd se usa el comando:
```bash
mix ecto.migrate
```

# Ecto Migraciones

Listar
```
 mix ecto.migrations
```

Salida ejemplo:
```
Repo: Auction.Repo

  Status    Migration ID    Migration Name
--------------------------------------------------
  down      20240824195538  create_items
```

Correr migraciones
```
mix ecto.migrate
```

Salida de ejemplo:
```
14:57:37.987 [info] == Running 20240824195538 Auction.Repo.Migrations.CreateItems.change/0 forward

14:57:37.989 [info] create table items

14:57:37.996 [info] == Migrated 20240824195538 in 0.0s
```

Devolver migraciones (rollback)

[Ecto Migration](https://hexdocs.pm/ecto_sql/Ecto.Migration.html)

[Ecto Rollback](https://hexdocs.pm/ecto_sql/Mix.Tasks.Ecto.Rollback.html)

```
mix ecto.rollback
```

O solo la anterior más reciente:
```
mix ecto.rollback --step 1
```

# Ecto Changesets

En Ecto actualizar no es tan simple como en Active Record. Para poder actualizar un registro en la base de datos hay que pasarle un `Ecto.Changeset` a `Ecto.Repo.update/2`.

Un `Ecto.Changeset` permite llevar cuenta de los cambios en un struct y además:

- validar datos de los cambios
- castear valores en los cambios
- definir restricciones (constraints) en los cambios

> A changeset allows you to verify that the data going into your database is valid.

Así que en vez de macros como en ActiveRecord, acá hay que escribir una función donde se ejecutan las validaciones.

```ruby
def changeset(item, params \\ %{}) do
    item
    |> cast(params, [:title, :description, :ends_at])
    |> validate_required(:title)
    |> validate_length(:title, min: 3)
    |> validate_length(:description, max: 200)
    |> validate_change(:ends_at, &validate/2)
end
```

Y así se ve cuando se prueba:
```bash
iex(7)> |> Auction.Item.changeset(
...(7)>   %{
...(7)>     title: "Prueba extensa",
...(7)>     description: "Soy una buena descripción descrita.",
...(7)>     ends_at: ~N[2024-12-25 00:00:00]
...(7)>   }
...(7)> )
#Ecto.Changeset<
  action: nil,
  changes: %{
    description: "Soy una buena descripción descrita.",
    title: "Prueba extensa",
    ends_at: ~U[2024-12-25 00:00:00Z]
  },
  errors: [],
  data: #Auction.Item<>,
  valid?: true,
  ...
>
```
