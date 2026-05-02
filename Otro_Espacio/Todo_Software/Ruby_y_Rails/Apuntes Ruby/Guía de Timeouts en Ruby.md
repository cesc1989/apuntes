# Guia de Timeouts en Ruby

Web: https://github.com/ankane/the-ultimate-guide-to-ruby-timeouts

Introducción:
> An unresponsive service can be worse than a down one. It can tie up your entire system if not handled properly. **All network requests should have a timeout.**

Dicen que:
> You should [avoid Ruby’s `Timeout` module](https://www.mikeperham.com/2015/05/08/timeout-rubys-most-dangerous-api/).

## Tipos de Timeout

- **connect (or open)** - time to open the connection
- **read (or receive)** - time to receive data after connected
- **write (or send)** - time to send data after connected
- **checkout** - time to checkout a connection from the pool
- **statement** - time to execute a database statement
- **lock (or acquisition)** - time to acquire a lock
- **request (or service)** - time to process a request
- **wait** - time to start processing a queued request
- **command** - time to run a command
- **solve** - time to solve an optimization problem


## Statement Timeouts

La guía los enfoca en los motores de bases de datos.

### En PostgreSQL

En `config/database.yml`:

```yaml
production:
  variables:
    statement_timeout: 5s # or ms, min, etc
```

Dice que se puede probar así:
```
SELECT pg_sleep(6);
```

Cuando se corran migraciones hay que subirle. Entonces la configuración puede leer el valor de una ENV:
```yaml
production:
  variables:
    statement_timeout: <%= ENV["STATEMENT_TIMEOUT"] || "5s" %>
```

Y usarse así:
```
STATEMENT_TIMEOUT=90s rails db:migrate
```