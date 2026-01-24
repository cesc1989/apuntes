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