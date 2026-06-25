# Apuntes Apex - Ciclo 005 - Solid Cache

## Configurando Solid Cache en Cash Flow

Relativamente fácil de instalar y configurar aunque cometí un error. Después de hacer los cambios y mandar a prod, me salía este error:

```ruby
ActiveRecord::StatementInvalid
Could not find table 'solid_cache_entries'
```

Cuando revisaba la carpeta de las bases de datos la de cache estaba ahí:
```bash
~/cashflow/app$ ll ../db/

-rw-r--r-- 1 ubuntu ubuntu 528384 Jun 24 19:09 cashflow_production.sqlite

-rw-r--r-- 1 ubuntu ubuntu  40960 Jun 24 19:09 cashflow_production_cache.sqlite

```

Y aún después de correr `bundle exec rails db:migrate` el error permanecía. Pasaron varias cosas para que se diera este error.

### Carpeta `db/cache_migrate` inexistente

Por alguna razón el comando no la crea. Solo creó el archivo del Schema. Claudio creó la migración en `db/cache_migrate` porque esa es la ruta especificada en el database.yml y corrigió la versión del schema para que coincidiera con la migración.

Ver commit: https://github.com/cesc1989/cashflow/commit/f6bb084314664e26f67c93f2cdb4e808c3c77783

### cache config debe ser `migrations_paths`

Había escrito a mano `migration_path`. Era en plural:
```
migrations_paths
```

### Solucionado

Una vez las dos cosas anteriores se liberaron Cash Flow quedó activo de nuevo. No sin antes correr estos comandos para corregir la base de datos de cache en el servidor:
```
rm /home/ubuntu/cashflow/db/cashflow_production_cache.sqlite*

RAILS_ENV=production bundle exec rails db:prepare
```

Uno para borrar y luego para recrear.

### Revisando cache funciona

Para comprobar que la caché funcionaba hice esto en la consola de Rails:
```ruby
Rails.cache.write("test_key", "hola mundo")

Rails.cache.read("test_key")
```

Luego pude comprobar en la base de datos que sí había registros:
```bash
sqlite3 /home/ubuntu/cashflow/db/cashflow_production_cache.sqlite "SELECT * FROM solid_cache_entries;"

1|production:rack::attack:29705469:req/ip:191.110.54.104||2026-06-24 19:09:35.468|5002972970595169618|213
2|production:rack::attack:29705473:req/ip:191.110.54.104||2026-06-24 19:13:19.624|-4423181243717394209|213
5|production:test_key||2026-06-24 19:15:13.839|-8663399159642937574|184
```