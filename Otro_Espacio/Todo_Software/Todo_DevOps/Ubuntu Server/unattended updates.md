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

```



# Revisando Configuración y Servicios

## Estado de nginx

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

