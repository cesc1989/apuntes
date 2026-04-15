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

## Errores 🐞

### Ignoring invalid environment assignment

Aún no doy para que con el mismo archivo puedan leer los tres servicios. El despliegue y el server corre con la configuración de arriba pero no Sidekiq. Veo este error al reiniciar el servicio:
```bash
-- Logs begin at Sat 2026-04-11 01:23:09 UTC, end at Wed 2026-04-15 13:58:28 UTC. --
Apr 15 13:30:00 localhost systemd[635]: enlacito.sidekiq.service: Scheduled restart job, restart counter is at 33670.
Apr 15 13:30:00 localhost systemd[635]: Stopped sidekiq.
Apr 15 13:30:00 localhost systemd[635]: enlacito.sidekiq.service: Ignoring invalid environment assignment 'export ENLACITO_REDIS_URL=redis://localhost:6379/1': /home/ubuntu/.enlacito.envs
Apr 15 13:30:00 localhost systemd[635]: enlacito.sidekiq.service: Ignoring invalid environment assignment 'export ENLACITO_RESEND_API_KEY=re_Prjax6mu_B2K5ZxCQSG2vm3s42RrK34qG': /home/ubuntu/.enlacito.envs
Apr 15 13:30:00 localhost systemd[635]: enlacito.sidekiq.service: Ignoring invalid environment assignment 'export ENLACITO_BOLD_API_KEY=1v1Ulfra6XfFg835vshKyfREnKJHOOvTQHo6onUExN0': /home/ubuntu/.enlacito.envs
Apr 15 13:30:00 localhost systemd[635]: enlacito.sidekiq.service: Ignoring invalid environment assignment 'export ENLACITO_APP_HOST=https://enlacito.co': /home/ubuntu/.enlacito.envs
Apr 15 13:30:00 localhost systemd[635]: Started sidekiq.
Apr 15 13:30:01 localhost enlacito_sidekiq[437919]: key not found: "ENLACITO_RESEND_API_KEY"
Apr 15 13:30:01 localhost enlacito_sidekiq[437919]: /home/ubuntu/enlacito/app/config/initializers/resend.rb:1:in 'fetch'
```

Según Claudio, al usar `export` en el archivo systemd no lo reconoce. Al actualizar el archivo y reinicio el servicio de sidekiq:
```bash
systemctl --user daemon-reload && systemctl --user restart enlacito.sidekiq.service
```

El servicio corre de nuevo. O sea que por ahí es.

Ahora queda resolver el tema con el despliegue y Passenger.

> [!Warning]
> Sobre el archivo que lee `EnvironmentFile`:
> > the file given in the `EnvironmentFile=` directive, is **not** a shell script. It is a simple list of assignments of the form variable=value. No `export`, no process substitution, no nothing. Just simple variable=value statements.
> >
> > De este comentario: https://unix.stackexchange.com/questions/766043/how-do-i-use-environment-variables-from-file-in-systemd#comment1460955_766048