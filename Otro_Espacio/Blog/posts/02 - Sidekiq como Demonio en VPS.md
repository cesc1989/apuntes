---
title: Configurar Sidekiq como demonio en un VPS
profileName: Otro Espacio Blog
postId: 5299
postType: post
categories:
  - 6652
---

Desde mis inicios en Ruby on Rails el application server que uso es Passenger. Siempre me pareció sencillo de usar y configurar porque se integra directo con Ngnix. Passenger es bueno pero hay muchas configuraciones que solo se pueden usar si se paga la licencia. Esta no es muy barata para proyectos hobby o empresas pequeñas. En ese caso la alternativa es Puma.

Puma es lo que viene por defecto con Rails. Para un proyecto quise probar esta alternativa. Así que ahí estaba yo configurando todo en el VPS. Cuando llega el momento de los despliegues y levantar el servicio con Puma me choqué con una pared. No logré hacerlo.

Eso fue hace muchos años atrás. Antes de la época de los LLMs así que tocaba bucear bastante en Internet. No tuve suerte así que dejé la idea y volví a Passenger. No pude lograr que Puma corriera como servicio de systemd.

## La definición del proceso

Pasaron los años y me tocó un evento similar pero con otra tecnología. Me tocaba poner a correr Sidekiq como demonio de systemd. Pensé que no iba a ser posible pero me decidí a hacerlo y hasta que al fin entendí más cosas de esta configuración.

Esta es la configuración para dejar una instancia de Sidekiq corriendo como demonio para una web que uso a modo personal:
```
# /home/ubuntu/.config/systemd/user/sidekiq.service

[Unit]
Description=sidekiq
After=syslog.target network.target

[Service]
Type=simple

WorkingDirectory=/home/ubuntu/cashflow/app

Environment=MALLOC_ARENA_MAX=2

EnvironmentFile=/home/ubuntu/.sidekiq_envs
ExecStart=/opt/rubies/ruby-3.1.0/bin/bundle exec sidekiq -e production

ExecReload=/usr/bin/kill -TSTP $MAINPID

RestartSec=1
Restart=on-failure

StandardOutput=syslog
StandardError=syslog

SyslogIdentifier=sidekiq

[Install]
WantedBy=default.target
```

Debe ubicarse en la carpeta `/home/ubuntu/.config/systemd/user/`.

Ya con eso definido y en la carpeta se activa con estos comandos:
```
systemctl --user daemon-reload
systemctl --user enable sidekiq.service
```

Verificamos que está corriendo con este otro:
```
systemctl --user status sidekiq.service
```

Salida de ejemplo:
```bash
systemctl --user status sidekiq
> ● sidekiq.service - sidekiq
> Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
> Active: inactive (dead)
> Jan 31 01:06:37 localhost systemd[3611161]: /home/ubuntu/.config/systemd/user/sidekiq.service:15: Executable "bundle" not found in path>
> Jan 31 01:06:37 localhost systemd[3611161]: sidekiq.service: Unit configuration has fatal error, unit will not be started.
```

## Detalles

El servicio toma el nombre del archivo. En este caso el archivo se llama `sidekiq.service` entonces ese será el nombre para tratar con el servicio.

Las variables de entorno se pueden definir en el mismo comando que arranca a Sidekiq así:
```
ExecStart=sh -c 'export SECRET_KEY_BASE=supermegahypersecretsecretosuperyeahmaracueya && /opt/rubies/ruby-3.1.0/bin/bundle exec sidekiq -e production'
```

Pero me gusta más la opción con directiva porque es más limpia. En el caso de:
```
EnvironmentFile=/home/ubuntu/.sidekiq_envs
```

`/home/ubuntu/.sidekiq_envs` es un archivo con las ENVs necesarias.

Y finalmente, tocó usar `Type=simple` porque, según entendí, el comando de inicio de Sidekiq se mantiene en el mismo proceso. No se separa.

## Recursos

La base con la que llegué a este configuración.

- [Run Sidekiq 6 as daemon in Production environment on Ubuntu 20.04](https://dev.to/kevinluo201/start-sidekiq-6-as-daemon-in-production-environment-on-ubuntu-20-04-4m7b)
- [Understanding Sidekiq's systemd Service Unit File](https://joelngwt.github.io/2023/12/02/understanding-sidekiq-systemd-service-unit-file.html)
- [Configuración de variables de entorno en systemd]([https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#Environment](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#Environment))