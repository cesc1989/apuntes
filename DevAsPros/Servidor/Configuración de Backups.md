# Configuración de Backups de BD y de Logs de Prod

Voy a cambiar el actual sistema que usa Mega como hospedaje porque tengo que tener la cuenta con la sesión iniciada y eso no me parece adecuado.

Pasaré a usar Cloudflare R2 como hospedaje. El script correrá un comando de Rclone para cargar los archivos en Cloudflare.

# Backups de la base de datos

## Mejoras al backup de sqlite

## Configuración de Cloudflare

Guías:
- Crear un API Token: https://developers.cloudflare.com/r2/api/tokens/

Cuando se crea el token se generan:

- token
- access key
- secret access key
- endpoint


## Configuración de rclone

Guías:
- Docs: https://rclone.org/downloads/
- Configurar rclone para Cloudflare: https://developers.cloudflare.com/r2/examples/rclone/

Se instala con:
```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

Verificación
```bash
which rclone
/usr/bin/rclone
```

### Prueba de rclone

Nombré el remote `cloudflare_r2`. Así que para leer los contenidos debo hacer:
```bash
rclone tree cloudflare_r2:coshinotes-backups
```

Posible respuesta:
```bash
/

0 directories, 0 files
```

### Problema de permisos

Cuando hago:
```bash
rclone lsd cloudflare_r2:
```

sale:
```bash
2026/01/24 01:07:20 ERROR : error listing: operation error S3: ListBuckets, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:07:20 NOTICE: Failed to lsd with 2 errors: last error was: operation error S3: ListBuckets, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied
```

También da error similar el script de subida del backup:
```bash
2026/01/24 01:05:02 ERROR : cashflowdb_20260124_010502.db.gz: Failed to copy: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:05:02 ERROR : Attempt 1/3 failed with 1 errors and: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:05:02 ERROR : cashflowdb_20260124_010502.db.gz: Failed to copy: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:05:02 ERROR : Attempt 2/3 failed with 1 errors and: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:05:03 ERROR : cashflowdb_20260124_010502.db.gz: Failed to copy: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:05:03 ERROR : Attempt 3/3 failed with 1 errors and: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied

2026/01/24 01:05:03 NOTICE: Failed to copy: failed to prepare upload: operation error S3: CreateBucket, https response error StatusCode: 403, RequestID: , HostID: , api error AccessDenied: Access Denied
```

##### Solución

Toca usar el comando rclone con la bandera `--s3-no-check-bucket` para que no dé ese error.

Esto porque creé los buckets manualmente. El comando normal sin este flag como que intenta crear el bucket o verificar que existe. Sin embargo, la API Token que creé solo tiene permisos para listar objetos en el bucket.


## Ejecución del script

Funciona!
```bash
 bash cashflow/app/scripts/backup_db.sh 
[INFO] 2026-01-24 01:37:24 - Iniciando backup de la base de datos...
[INFO] 2026-01-24 01:37:24 - Base de datos encontrada: 468K
[INFO] 2026-01-24 01:37:24 - Creando backup con SQLite...
[INFO] 2026-01-24 01:37:24 - ✓ Backup SQLite creado: /home/ubuntu/cashflow/backups/cashflowdb_20260124_013724.db
[INFO] 2026-01-24 01:37:24 - Comprimiendo backup...
[INFO] 2026-01-24 01:37:24 - ✓ Backup comprimido: /home/ubuntu/cashflow/backups/cashflowdb_20260124_013724.db.gz
[INFO] 2026-01-24 01:37:24 - Tamaño: 128K
[INFO] 2026-01-24 01:37:24 - Subiendo a Cloudflare R2...
[INFO] 2026-01-24 01:37:25 - ✓ Backup subido exitosamente a Cloudflare R2
[INFO] 2026-01-24 01:37:25 - Ruta: cloudflare_r2:cashflow-backups/databases/cashflowdb_20260124_013724.db.gz
[INFO] 2026-01-24 01:37:25 - Verificando subida...
[INFO] 2026-01-24 01:37:26 - ✓ Verificación exitosa: archivo encontrado en R2
[INFO] 2026-01-24 01:37:26 - Limpiando archivos temporales...
[INFO] 2026-01-24 01:37:26 - Limpieza completada
=========================================
[INFO] 2026-01-24 01:37:26 - ✅ BACKUP COMPLETADO EXITOSAMENTE
[INFO] 2026-01-24 01:37:26 - Base de datos: cashflow_production.sqlite
[INFO] 2026-01-24 01:37:26 - Backup creado: cashflowdb_20260124_013724.db.gz
[INFO] 2026-01-24 01:37:26 - Destino: Cloudflare R2 (cloudflare_r2:cashflow-backups/databases/)
[INFO] 2026-01-24 01:37:26 - Fecha: Sat Jan 24 01:37:26 UTC 2026
=========================================
```