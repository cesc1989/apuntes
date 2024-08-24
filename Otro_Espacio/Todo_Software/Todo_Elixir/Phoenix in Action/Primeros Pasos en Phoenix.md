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

# Ecto

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

# Migraciones

Listar
```
 mix ecto.migrations

Repo: Auction.Repo

  Status    Migration ID    Migration Name
--------------------------------------------------
  down      20240824195538  create_items
```

Correr migraciones
```
$ mix ecto.migrate

14:57:37.987 [info] == Running 20240824195538 Auction.Repo.Migrations.CreateItems.change/0 forward

14:57:37.989 [info] create table items

14:57:37.996 [info] == Migrated 20240824195538 in 0.0s
```