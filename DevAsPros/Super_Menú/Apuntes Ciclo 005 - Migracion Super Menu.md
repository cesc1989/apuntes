# Apuntes Ciclo 005 - Migración a Host Hatch

## Pasos

Resumen de los pasos:
1. Crear carpetas
2. Configurar y hacer pull desde el VPS
3. Configurar secrets en GH
	1. KNONW_HOSTS
	2. TARGET_HOST
4. Actualizar registro A en Namecheap
5. Instalar certbot
6. Corregir configuración de chruby
7. Definir las ENVs
	1. De AWS por carga de imágenes y el CDN
8. Configurar el certificado SSL
9. Comprobaciones de funcionalidad

# Detalles de la Migración

## Descargar la base de datos con scp

Aprovechando que tengo definidos las configuraciones en `~/.ssh/config`. Así copió del servidor en Linode a local:
```bash
scp dap_node:/home/ubuntu/supermenu/db/supermenu_production.sqlite ~/Downloads/supermenu_production.sqlite
```

Y así cargo al servidor en Host Hatch:
```bash
scp ~/Downloads/supermenu_production.sqlite dap_hatch:/home/ubuntu/supermenu/db/supermenu_production.sqlite
```

## Desactivar servicio en dap_node

Hay que apagar el proceso de passenger para esta aplicación. ¿Hay que hacer algo o solo basta con borrar los archivos y el host de nginx?