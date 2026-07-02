# Apuntes Apex - Ciclo 006 - Solid Cable

## Configurando solid dable en Cash Flow

Esta vez ya está más claro el proceso. Después de instalar la gema y correr el comando configuré prod y development teniendo en cuenta que en el archivo `config/database.yml` el atributo se llama `migrations_paths`.

Para poder probar en local dejé así la configuración:
```yaml
development:
  primary:
    <<: *default
    database: db/cashflow_development.sqlite
  cable:
    <<: *default
    database: db/cashflow_cable.sqlite
    migrations_paths: db/cable_migrate

production:
  primary:
    <<: *default
    database: /home/ubuntu/cashflow/db/cashflow_production.sqlite
  cache:
    <<: *default
    database: /home/ubuntu/cashflow/db/cashflow_production_cache.sqlite
    migrations_paths: db/cache_migrate
  cable:
    <<: *default
    database: /home/ubuntu/cashflow/db/cashflow_cable.sqlite
    migrations_paths: db/cable_migrate
```

Corrí `./bin/rails db:prepare` y vi la base de datos creada.

### Comprobaciones en Development

En desarrollo comprobé con los siguientes comandos.

Que existe la tabla `solid_cable_messages`:
```bash
sqlite3 db/cashflow_cable.sqlite ".tables"
```

Salida:
```
ar_internal_metadata  schema_migrations     solid_cable_messages
```

La configuración de Solid Cable hace que los mensajes se limpien enseguida. La gracia no es conservar si no que funcione como cola. Entonces este comando:
```bash
sqlite3 db/cashflow_cable.sqlite "SELECT * FROM solid_cable_messages;"
```

No devuelve nada.

Una forma de confirmar que se está usando el adaptador de Solid Cable es correr esto en una consola de Rails:
```ruby
ActionCable.server.pubsub.class
```

La salida:
```ruby
ActionCable::SubscriptionAdapter::SolidCable
```

### Comprobaciones en Production

Para prod son estos dos.

```bash
sqlite3 /home/ubuntu/cashflow/db/cashflow_cable.sqlite ".tables"
```

```bash
sqlite3 /home/ubuntu/cashflow/db/cashflow_cable.sqlite "SELECT * FROM solid_cable_messages;"
```

## Desinstalando Redis en Host Hatch

Como Super Menu ni Postlane necesitan Redis (ni lo van a necesitar porque usaré Solid Trifecta) procedí a quitarlo.

Estos son los comandos:

```bash
redis-cli ping                 # debe responder PONG si está corriendo
redis-cli info clients         # ver conexiones activas

sudo systemctl stop redis-server
sudo systemctl disable redis-server
sudo systemctl status redis-server   # confirma: inactive (dead)

sudo apt remove --purge redis-server redis-tools
sudo apt autoremove

sudo rm -f /etc/apt/sources.list.d/redis.list
sudo apt update

sudo rm -rf /etc/redis
sudo rm -rf /var/lib/redis
sudo rm -rf /var/log/redis
```