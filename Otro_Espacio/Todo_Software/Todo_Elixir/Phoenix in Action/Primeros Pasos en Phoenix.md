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