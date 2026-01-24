# Configuración de Backups de BD y de Logs de Prod

Voy a cambiar el actual sistema que usa Mega como hospedaje porque tengo que tener la cuenta con la sesión iniciada y eso no me parece adecuado.

Pasaré a usar Cloudflare R2 como hospedaje. El script correrá un comando de Rclone para cargar los archivos en Cloudflare.

## Backups de la base de datos

### Mejoras al backup de sqlite

### Configuración de Cloudflare

Guías:
- Crear un API Token: https://developers.cloudflare.com/r2/api/tokens/

Cuando se crea el token se generan:

- token
- access key
- secret access key
- endpoint


### Configuración de rclone

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

#### Prueba de rclone

Nombré el remote `cloudflare_r2`. Así que para leer los contenidos debo hacer:
```bash
rclone tree cloudflare_r2:coshinotes-backups
```

Posible respuesta:
```bash
/

0 directories, 0 files
```