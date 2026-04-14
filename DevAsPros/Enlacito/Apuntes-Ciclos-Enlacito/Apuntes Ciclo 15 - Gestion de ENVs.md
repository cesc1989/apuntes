# Apuntes Ciclo 15 - Gestión de ENVs

Dado a que hay (y habrá) varias aplicaciones Rails en el VPS veo necesario que las ENVs de cada una están prefijadas. De esa manera puedo identificar y diferenciarlas unas de otras. Saber cuáles se pueden borrar y cuáles hacen falta.

Además, varios procesos necesitan leer las ENVs de cada aplicación:

- El despliegue
- Sidekiq (si lo usa)
- Passenger (Rails server)

Será más cómodo tenerlas todas en un solo archivo por aplicación y luego solo referenciar ese archivo.

## El Archivo

Creé el archivo `~/.enlacito.envs` que tiene las variables:
```bash
SECRET_KEY_BASE="CHANGEME"

export ENLACITO_REDIS_URL="redis://127.0.0.1:6379/1"
export ENLACITO_RESEND_API_KEY="CHANGEME"

export ENLACITO_BOLD_API_KEY="CHANGEME"
export ENLACITO_APP_HOST="CHANGEME"
```

Está en el repo para saber que existe y que debe crearse en el VPS.

## Uso en scripts de despliegues

Este fue el cambio que hice en los archivos `scripts/002_pull_repo.sh`, `scripts/003_after_deploy.sh` y `scripts/004_application_start.sh`:
```diff
- . /home/ubuntu/.profile
+ . /home/ubuntu/.enlacito.envs
```

## Uso para Sidekiq

Esto fue lo que cambió para `scripts/enlacito.sidekiq.service`:
```diff
- EnvironmentFile=/home/ubuntu/.enlacito.sidekiq.envs
+ EnvironmentFile=/home/ubuntu/.enlacito.envs
```

## Uso para Passenger (rails server)

Passenger lee las variables de Nginx, así que las lee del `~/.profile`. Por lo tanto, y como quería quitar estas envs de ese archivo, toca referenciarlo así:
```~/.profile
# (...)

. /home/ubuntu/.enlacito.envs
```