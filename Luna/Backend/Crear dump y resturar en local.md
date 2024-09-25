# Comandos para crear un dump de PostgreSQL y cargar en BD local

## Crear dump con pg_dump

El comando base es este:
```bash
pg_dump -h [host_name] -U [user_name] -f /path/to/destination/filename [db_name]
```

Ejemplo:
```bash
pg_dump -h dev-francisco-alpha-edge-09-23t03-40.ctvhkhbiykgu.us-west-2.rds.amazonaws.com -U luna_api -f ~/Downloads/dump_alpha luna_api_staging
```

## Restaurar

Como hice el dump con el modo `-f`, hay que restaurar cargando el archivo mediante `psql`.

Comando base:
```bash
psql [db_name] --host [host_name] --username [username] < /path/to/dump_file
```

Ejemplo:
```bash
psql luna_api_development --host localhost --username francisco < ~/Downloads/dump_alpha
```

> [!WARNING]
> La base de datos objetivo debe estar recién creada. Si ya existe, hay que borrarla y recrearla.