# Unattended Updates in Ubuntu Server

Quiero configurar Unattended Upgrades para el servidor de DevAsPros.

Algunos datos:

- Ubuntu 20.04.6 LTS

# Pasos y Comandos

Hay varias guías pero esto que tengo en [un gist](https://gist.github.com/cesc1989/b4c685b6ca41f777949780c9729a3b70) sigue siendo válido y se resume en lo que tengo en [backend stuff](https://github.com/cesc1989/backendstuff/tree/master/configurators/unattended-upgrades).

## Configuración Inicial

Actualiza la lista de paquetes (fuentes) e instala:
```
sudo apt update && sudo apt upgrade
```

Instala el paquete unattended-upgrades (normalmente ya está instalado):
```
sudo apt-get install -y unattended-upgrades
```

Activa la configuración (selecciona Yes):
```
sudo dpkg-reconfigure unattended-upgrades
```

Se configuran dos archivos:

- `/etc/apt/apt.conf.d/20auto-upgrades`
- `/etc/apt/apt.conf.d/50unattended-upgrades`

El primero indica la frecuencia en que unattended-upgrades hará lo suyo: descargar paquetes, instalar, remover basura.

El segundo hará cosas más de cuidado como qué tipo de cosas a instalar (seguridad, por ejemplo), enviar emails y activar el reiniciado automático.

La configuración de la frecuencia que describo en el [gist](`/etc/apt/apt.conf.d/50unattended-upgrades`) y en [backend stuff](https://github.com/cesc1989/backendstuff/blob/master/configurators/unattended-upgrades/20auto-upgrades) es la misma que seguí para el servidor de DevAsPros:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "3";
APT::Periodic::AutocleanInterval "9";
```

Donde cada número indica la frecuencia en días para que el servidor haga lo suyo.

## Reinicio Automático

Para que el servidor se reinicie de manera automática y envíe correos sobre el estado de las actualizaciones hay que cambiar algunos ajustes en `/etc/apt/apt.conf.d/50unattended-upgrades`.

Para indicar a qué correo enviar mensajes se cambia esta configuración:
```
Unattended-Upgrade::Mail "";
```

Para indicar que se reinicie el servidor por si mismo se cambia:
```
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "19:00"; // Optional
```

> Para que el reinicio funcione se necesita el paquete **update-notifier-common**.

## Envío de Correos

En [el gist](https://gist.github.com/cesc1989/b4c685b6ca41f777949780c9729a3b70#email-notification-configuration) hay instrucciones pero no pude instalar el software `heirloom-mailx`.

Hay que buscar una forma actual.

# Probando Configuración de unattended-upgrades

Se prueba con este comando:
```
sudo unattended-upgrade -v -d --dry-run
```

Y se puede ejecutar manualmente con este otro:
```
sudo unattended-upgrade -v -d
```

Así me fue cuando probé:
```
ubuntu@localhost:~$ sudo unattended-upgrade -v -d
[sudo] password for ubuntu: 
Running on the development release
Starting unattended upgrades script
Allowed origins are: o=Ubuntu,a=focal, o=Ubuntu,a=focal-security, o=UbuntuESMApps,a=focal-apps-security, o=UbuntuESM,a=focal-infra-security
Initial blacklist: 
Initial whitelist (not strict):

(...)

pkgs that look like they should be upgraded: 
Fetched 0 B in 0s (0 B/s)

fetch.run() result: 0
Packages blacklist due to conffile prompts: []
No packages found that can be upgraded unattended and no pending auto-removals
Package dmeventd has a higher version available, checking if it is from an allowed origin and is not pinned down.
Package dmsetup has a higher version available, checking if it is from an allowed origin and is not pinned down.
Package libdevmapper-event1.02.1 has a higher version available, checking if it is from an allowed origin and is not pinned down.
Package libdevmapper1.02.1 has a higher version available, checking if it is from an allowed origin and is not pinned down.
Package liblvm2cmd2.03 has a higher version available, checking if it is from an allowed origin and is not pinned down.
Package lvm2 has a higher version available, checking if it is from an allowed origin and is not pinned down.
Extracting content from /var/log/unattended-upgrades/unattended-upgrades-dpkg.log since 2024-09-03 18:43:47

No /usr/bin/mail or /usr/sbin/sendmail, can not send mail. You probably want to install the mailx package.
```

> Nota: me dice que instale `mailx`


# Revisando Configuración y Servicios

## Estado de nginx

Con el comando:
```
sudo service nginx status
```

Resultado:
```
ubuntu@localhost:~$ sudo service nginx status
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2024-09-01 15:48:08 UTC; 2 days ago
       Docs: man:nginx(8)
   Main PID: 4038606 (nginx)
      Tasks: 72 (limit: 1126)
     Memory: 372.8M
     CGroup: /system.slice/nginx.service
             ├─4038606 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─4053731 Passenger watchdog
             ├─4053736 Passenger core
             ├─4053744 nginx: worker process is shutting down
             ├─4055483 Passenger RubyApp: /home/ubuntu/cashflow/app (production)
             ├─4127484 Passenger RubyApp: /home/ubuntu/coshinotes/app (production)
             ├─4147523 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─4147524 Passenger watchdog
             ├─4147528 Passenger core
             ├─4147539 nginx: worker process
             ├─4159768 Passenger RubyApp: /home/ubuntu/cashflow/app (production)
             └─4159888 Passenger RubyApp: /home/ubuntu/coshinotes/app (production)

Sep 01 15:48:08 localhost systemd[1]: Starting A high performance web server and a reverse proxy server...
Sep 01 15:48:08 localhost systemd[1]: Started A high performance web server and a reverse proxy server.
```

## Estado de Sidekiq

Con el comando:
```
systemctl --user status sidekiq.service
```

Resultado:
```
ubuntu@localhost:~$ systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2024-09-01 15:48:08 UTC; 2 days ago
   Main PID: 4038621 (bundle)
     CGroup: /user.slice/user-1000.slice/user@1000.service/sidekiq.service
             └─4038621 sidekiq 6.5.12 app [0 of 1 busy]

Sep 01 15:48:08 localhost systemd[3611161]: Started sidekiq.
Sep 01 15:48:10 localhost sidekiq[4038621]: 2024-09-01T15:48:10.043Z pid=4038621 tid=2ewf5 INFO: Booting Sidekiq 6.5.12 with Sidekiq::RedisConnection::RedisAdapter options {:url=>nil}
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.084Z pid=4038621 tid=2ewf5 INFO: Booted Rails 7.0.4 application in production environment
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.085Z pid=4038621 tid=2ewf5 INFO: Running in ruby 3.1.0p0 (2021-12-25 revision fb4df44d16) [x86_64-linux]
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.085Z pid=4038621 tid=2ewf5 INFO: See LICENSE and the LGPL-3.0 for licensing details.
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.085Z pid=4038621 tid=2ewf5 INFO: Upgrade to Sidekiq Pro for more features and support: https://sidekiq.org
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.102Z pid=4038621 tid=2ewf5 INFO: Loading Schedule
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.102Z pid=4038621 tid=2ewf5 INFO: Scheduling recurring_expenditures {"class"=>"RecurringExpendituresWorker", "cron"=>"0 0 1 * *", "queue"=>"default"}
Sep 01 15:48:11 localhost sidekiq[4038621]: 2024-09-01T15:48:11.114Z pid=4038621 tid=2ewf5 INFO: Schedules Loaded
```