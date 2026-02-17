# Apuntes Ciclo 005 - Migración a Host Hatch

Etiquetas: #despliegue_vps 

## Todos los Pasos

Resumen de los pasos:
1. Crear carpetas
2. Configurar y hacer pull desde el VPS
3. Configurar secrets en GH
	1. `KNONW_HOSTS`
	2. `TARGET_HOST`
4. Actualizar registro A en Namecheap
5. Instalar certbot
6. Copiar base de datos entre VPS
7. Corregir configuración de chruby
8. Definir las ENVs
	1. De AWS por carga de imágenes y el CDN
9. Configurar el certificado SSL
10. Comprobaciones de funcionalidad
11. Desactivar todo lo del dap_node
	1. Borrar carpeta supermenu
	2. Borrar ENVs correspondientes

# Detalles de la Migración

## Crear las carpetas del proyecto en el VPS

Con este comando:
```bash
mkdir -p ~/supermenu/{app,deployments/{api-gems,api-release,logs},backups,db}
```

## Configurar y hacer pull desde el VPS

Tuve que cargar la llave privada que está configurada en GitHub. Luego de eso hay que activar el agente ssh y cargar la llave privada.

Ver [[Apuntes_Ciclo_02_-_Configuración_-_Cash_Flow#Configuración para hacer pull del repo GitHub desde el VPS]]

## Configurar secrets en el repo en GitHub

Hay que modificar los valores previos de los secretos `TARGET_HOST` y `KNOWN_HOSTS`.

El primero es solo actualizar poniendo la IP del nuevo VPS.

Para el segundo la clave está en copiar los valores que estén en `~/.ssh/known_hosts`. Hay que copiar las líneas que tengan la IP del VPS. O se borran, se hace login desde local y luego se copian esas líneas.

## Instalar cerbot

> [!Note]
> Esto toca agregarlo a los scripts de configuración del VPS.

Ver [[Apuntes_Ciclo_04_-_Post_Despliegue_-_Cash_Flow#Instalar certificados SSL con Certbot]]

## Copiar la base de datos entre VPS

Aprovechando que tengo definidos las configuraciones en `~/.ssh/config`. Así copió del servidor en Linode a local:
```bash
scp dap_node:/home/ubuntu/supermenu/db/supermenu_production.sqlite ~/Downloads/supermenu_production.sqlite
```

Y así cargo al servidor en Host Hatch:
```bash
scp ~/Downloads/supermenu_production.sqlite dap_hatch:/home/ubuntu/supermenu/db/supermenu_production.sqlite
```

## Corregir configuración de chruby

Fijarse que en el archivo `~/.profile` se configure chruby de esta forma:
```bash
if [ -n "$BASH_VERSION" ]; then
   source /usr/local/share/chruby/chruby.sh && chruby 3.2.5
fi
```

## Configurar el certificado SSL

Ver [[Apuntes_Ciclo_04_-_Post_Despliegue_-_Cash_Flow#Uso de Certbot para generar certificados]]

Me tocó desactivar la sección SSL del archivo nginx porque sino salía otra cosa en certbot. Lo dejé solo con el bloque http (puerto 80) y luego sí pude hacer el proceso como en el documento enlazado.

```bash
Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: supermenu.devaspros.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Requesting a certificate for supermenu.devaspros.com

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/supermenu.devaspros.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/supermenu.devaspros.com/privkey.pem
This certificate expires on 2026-05-18.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for supermenu.devaspros.com to /etc/nginx/sites-enabled/nginx.supermenu.production.conf
Congratulations! You have successfully enabled HTTPS on https://supermenu.devaspros.com
```

## Desactivar servicio en dap_node

Hay que apagar el proceso de passenger para esta aplicación. ¿Hay que hacer algo o solo basta con borrar los archivos y el host de nginx?