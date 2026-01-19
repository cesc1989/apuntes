# Sidekiq como Demonio

Etiquetas: #sidekiq_cron 

Relacionado [[Apuntes_Ciclo_17_-_Sidekiq_-_Cash_Flow]]

## Cómo ver el estado cuando hay varios servicios Sidekiq en el mismo servidor

Para el caso de Cashflow se ven así:
```bash
systemctl --user status sidekiq.service
```

Para el caso de Enlacito se ven así:
```bash
systemctl --user status enlacito.sidekiq.service
```

# Configuración de Redis y Sidekiq en un mismo servidor para diferentes aplicaciones

Cuando terminé de configurar Sidekiq para enlacito y fui a la versión web me di cuenta que compartía la misma base de datos Redis con CashFlow.

Me di cuenta porque pude ver que tenían los mismo trabajos en "Muerto" y la misma conexión: `redis://localhost:6379/0`.

Eso no está bien. Son aplicaciones diferentes y no deben compartir la misma cola ni la misma interfaz web. Tenía que separarlos.

## Cambiar la base de datos de Redis

Según DeepSeek, una instancia de Redis tiene 15 bases de datos por defecto. Así que para lograr la separación podría indicar una base de datos diferente por aplicación.

Eso hice.

### Cambios en CashFlow

En CashFlow apunte a la base de datos cero (0).

En el `.profile` del servidor:
```bash
export REDIS_URL="redis://localhost:6379"
```

En `sidekiq.rb`:
```ruby
url: ENV["REDIS_URL"],
```

En el archivo de ENVs para el demonio de Sidekiq en el servidor:
```bash
REDIS_URL="redis://localhost:6379/0"
```

> [!Note]
> Este archivo y ENV deberán pasar a llamarse con un sufijo de la aplicación. Para poder diferenciarlas de otras envs.

### Cambios en Enlacito

En Enlacito apunte a la base de datos cero (1).

En el `.profile` del servidor:
```bash
export REDIS_URL_ENLACITO="redis://localhost:6379/1"
```

En `sidekiq.rb`:
```ruby
url: ENV.fetch("REDIS_URL_ENLACITO", "redis://127.0.0.1:6379/1"),
```

En el archivo de ENVs para el demonio de Sidekiq en el servidor:
```bash
REDIS_URL_ENLACITO="redis://localhost:6379/1"
```