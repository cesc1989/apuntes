# Unattended Upgrades in Ubuntu Server

Quiero configurar Unattended Upgrades para el servidor de DevAsPros.

Algunos datos:
- Ubuntu 20.04.6 LTS
- Software involucrado:
	- unattended-upgrades
	- update-notifier-common
	- bsd-mailx
		- Instala Postfix para usar Mailx

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

La configuración de la frecuencia que describo en el [gist](https://gist.github.com/cesc1989/b4c685b6ca41f777949780c9729a3b70) y en [backend stuff](https://github.com/cesc1989/backendstuff/blob/master/configurators/unattended-upgrades/20auto-upgrades) es la misma que seguí para el servidor de DevAsPros:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
```

Donde cada número indica la frecuencia en días para que el servidor haga lo suyo.

### Explicación de las opciones

- `APT::Periodic::Update-Package-Lists "1"`
	- Actualiza la lista de paquetes a diario. **Este es importante** para que siempre que se vayan a hacer las actualizaciones, se usen las fuentes más recientes.
- `APT::Periodic::Download-Upgradeable-Packages "1"`
	- Descarga las actualizaciones a diario. Así no tienen que descargarse todas al tiempo.
- `APT::Periodic::Unattended-Upgrade "1"`
	- Ejecuta las actualizaciones a diario.
- `APT::Periodic::AutocleanInterval "7"`
	- Limpia la cache de paquetes cada 7 días.

Para el servidor de DevAsPros, inicialmente había configurado `APT::Periodic::Unattended-Upgrade "1"` para cada 3 días pero decidí cambiarlo a diario para seguir el principio de "fail fast". Si va a salir algo mal en este proceso, quiero verlo ya y resolverlo pronto.

Así también me acostumbro a lidiar con esta configuración y la entiendo a fondo.

## Reinicio Automático del Servidor

Para que el servidor se reinicie de manera automática y envíe correos sobre el estado de las actualizaciones hay que cambiar algunos ajustes en `/etc/apt/apt.conf.d/50unattended-upgrades`.

Para indicar a qué correo enviar mensajes se cambia esta configuración:
```
Unattended-Upgrade::Mail "";
```

Para indicar que se reinicie el servidor por si mismo se cambia:
```
Unattended-Upgrade::Automatic-Reboot "true";

Unattended-Upgrade::Automatic-Reboot-Time "06:00"; // 06am UTC is 01am UTC-05
```

> Para que el reinicio funcione se necesita el paquete `update-notifier-common`.

## Envío de Correos

En [el gist](https://gist.github.com/cesc1989/b4c685b6ca41f777949780c9729a3b70#email-notification-configuration) hay instrucciones pero no pude instalar el software `heirloom-mailx`.

Eso de `heirloom-mailx` como que no existe. Entonces [encontré](https://vernon.wenberg.net/linux/set-up-unattended-upgrades-on-ubuntu-20-04/) que se configura `mailx` así:
```
sudo apt install bsd-mailx
```

Que instala todo esto:
```
The following additional packages will be installed:
  liblockfile-bin liblockfile1 postfix ssl-cert
Suggested packages:
  procmail postfix-mysql postfix-pgsql postfix-ldap postfix-pcre postfix-lmdb postfix-sqlite sasl2-bin | dovecot-common resolvconf postfix-cdb postfix-doc openssl-blacklist
The following NEW packages will be installed:
  bsd-mailx liblockfile-bin liblockfile1 postfix ssl-cert
```

### Configuración de Postfix con Gmail

Encontré una guía en [Linode](https://www.linode.com/docs/guides/configure-postfix-to-send-mail-using-gmail-and-google-workspace-on-debian-or-ubuntu/).

En este archivo: `/etc/postfix/main.cf`
```
myhostname = devaspros.com
```

> Probé así pero no sé si eso funciona.
>
> NOTA: Las guías de Linode dicen que ellos pueden bloquear los puertos de correo para cuentas nuevas.

Abre o crea este archivo: `/etc/postfix/sasl/sasl_passwd`

Y agrega las credenciales de la cuenta de Gmail:
```
[smtp.gmail.com]:587 cuenta@gmail.com:clave
```

Luego lanzamos este comando para que se genere un hash db para Postfix:
```
sudo postmap /etc/postfix/sasl/sasl_passwd
```

Finalmente, aseguramos este archivo porque es texto plano:
```
sudo chown root:root /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db

sudo chmod 0600 /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
```

Para terminar la configuración, abrimos de nuevo `/etc/postfix/main.cf` y añadimos estas líneas:
```
relayhost = [smtp.gmail.com]:587

# Enable SASL authentication
smtp_sasl_auth_enable = yes

# Disallow methods that allow anonymous authentication
smtp_sasl_security_options = noanonymous

# Location of sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd

# Enable STARTTLS encryption
smtp_tls_security_level = encrypt

# Location of CA certificates
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

> NOTA: `smtp_tls_security_level` puede estar duplicada así que quita las anteriores.

Reinicia Postfix para que tome las configuraciones:
```
sudo systemctl restart postfix
```

### Probar Envío de Correo desde el Servidor

Se prueba con el comando `sendmail`:
```
sendmail frajaquico@gmail.com
From: devaspros@gmail.com
Subject: Prueba Servidor
Correo de prueba desde el servidor
.
```

Se escribe la primera línea y se presiona enter. No pasa nada. Hay que seguir escribiendo el resto de líneas. El punto + enter envía el correo y sale del editor.

Verifica en los logs que el correo esté saliendo:
```
sudo tail -f /var/log/syslog
```

Pero estoy viendo esto:
```
Sep  4 23:49:54 localhost kernel: [36276332.471271] [UFW BLOCK] IN=eth0 OUT= MAC=f2:3c:93:9d:4c:60:fe:ff:ff:ff:ff:ff:08:00 SRC=110.183.54.49 DST=139.144.196.192 LEN=40 TOS=0x00 PREC=0x00 TTL=41 ID=31503 PROTO=TCP SPT=40646 DPT=23 WINDOW=36054 RES=0x00 SYN URGP=0 
Sep  4 23:50:04 localhost kernel: [36276342.373403] [UFW BLOCK] IN=eth0 OUT= MAC=f2:3c:93:9d:4c:60:fe:ff:ff:ff:ff:ff:08:00 SRC=162.216.150.232 DST=139.144.196.192 LEN=44 TOS=0x00 PREC=0x60 TTL=241 ID=54321 PROTO=TCP SPT=53048 DPT=40007 WINDOW=65535 RES=0x00 SYN URGP=0 
Sep  4 23:50:11 localhost kernel: [36276349.825829] [UFW BLOCK] IN=eth0 OUT= MAC=f2:3c:93:9d:4c:60:fe:ff:ff:ff:ff:ff:08:00 SRC=206.168.34.168 DST=139.144.196.192 LEN=60 TOS=0x00 PREC=0x00 TTL=44 ID=54552 PROTO=TCP SPT=42868 DPT=81 WINDOW=42340 RES=0x00 SYN URGP=0 

Sep  4 23:50:12 localhost postfix/smtp[26400]: connect to smtp.gmail.com[2607:f8b0:4004:c08::6d]:587: Connection timed out

Sep  4 23:50:33 localhost kernel: [36276371.390790] [UFW BLOCK] IN=eth0 OUT= MAC=f2:3c:93:9d:4c:60:fe:ff:ff:ff:ff:ff:08:00 SRC=83.222.190.66 DST=139.144.196.192 LEN=40 TOS=0x00 PREC=0x00 TTL=231 ID=21340 PROTO=TCP SPT=41225 DPT=2158 WINDOW=1024 RES=0x00 SYN URGP=0 

Sep  4 23:50:42 localhost postfix/smtp[26400]: connect to smtp.gmail.com[172.253.63.109]:587: Connection timed out

Sep  4 23:50:42 localhost postfix/smtp[26400]: 3449C23E79: to=<frajaquico@gmail.com>, relay=none, delay=79, delays=19/0.01/60/0, dsn=4.4.1, status=deferred (connect to smtp.gmail.com[172.253.63.109]:587: Connection timed out)

Sep  4 23:50:44 localhost kernel: [36276382.866615] [UFW BLOCK] IN=eth0 OUT= MAC=f2:3c:93:9d:4c:60:fe:ff:ff:ff:ff:ff:08:00 SRC=92.63.197.192 DST=139.144.196.192 LEN=44 TOS=0x00 PREC=0x00 TTL=238 ID=54116 PROTO=TCP SPT=53012 DPT=31127 WINDOW=1025 RES=0x00 SYN URGP=0
```

Voy a pedir en soporte que me habiliten el puerto 587.



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

# Ejecución de Unattended-upgrades

Luego de que cambiara todo a un día e inicié sesión en el servidor vi algunas cosas.

## No se reinició el servidor

Tal vez fue cosa de la hora. Hoy, 5 de Septiembre, el servidor aún pedía reiniciada:
```
Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

4 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm

New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


*** System restart required ***
You have mail.
Last login: Thu Sep  5 02:22:25 2024 from 191.110.58.7
```

### Comprobar el uptime del servidor para saber si se reinició

Existe el comando `uptime` para saber si el tiempo que lleva arriba el servidor.

```
$ uptime
12:06:29 up 420 days,  9:03,  1 user,  load average: 0.10, 0.08, 0.03
```

Más sobre este comando en [este artículo](https://linuxhandbook.com/uptime-command/).

## Mensaje You have mail

Noté ese mensaje. ¿Cómo reviso dicho correo? Lo puedo hacer de varias formas.

La más directa es uno de estos comandos:
```
sudo cat /var/spool/mail/root

sudo cat /var/spool/mail/ubuntu
```

En mí caso, debo usar el segundo comando para el usuario "ubuntu". Puedo ver algunas cosas
```
This is the mail system at host devaspros.com.

Enclosed is the mail delivery report that you requested.

                   The mail system

<frajaquico@gmail.com>: connect to smtp.gmail.com[172.253.63.108]:587:
    Connection timed out

--0DB8D23E73.1725491958/devaspros.com
Content-Description: Delivery report
Content-Type: message/delivery-status

(...)

Final-Recipient: rfc822; frajaquico@gmail.com
Original-Recipient: rfc822;frajaquico@gmail.com
Action: delayed
Status: 4.4.1
Diagnostic-Code: X-Postfix; connect to smtp.gmail.com[172.253.63.108]:587:
    Connection timed out
```

¿Será algo del bloqueo de puertos de Linode?

> Puedo también ver la lista con el [comando](https://superuser.com/questions/306163/what-is-the-you-have-new-mail-message-in-linux-unix) `mail` y luego presionando el número del correo de la lista.

## Comprobando que se esté ejecutando unattended-upgrades

¿Cómo sé si está corriendo y haciendo su trabajo? Hay un par de formas.

Tail al log
```
sudo tail -n 30 /var/log/unattended-upgrades/unattended-upgrades.log

2024-09-05 06:04:33,422 INFO Starting unattended upgrades script
2024-09-05 06:04:33,425 INFO Allowed origins are: o=Ubuntu,a=focal, o=Ubuntu,a=focal-security, o=UbuntuESMApps,a=focal-apps-security, o=UbuntuESM,a=focal-infra-security
2024-09-05 06:04:33,425 INFO Initial blacklist: 
2024-09-05 06:04:33,425 INFO Initial whitelist (not strict): 
2024-09-05 06:04:35,315 INFO Packages that will be upgraded: apparmor libapparmor1 python3-twisted python3-twisted-bin
2024-09-05 06:04:35,316 INFO Writing dpkg log to /var/log/unattended-upgrades/unattended-upgrades-dpkg.log
2024-09-05 06:04:49,080 INFO All upgrades installed
2024-09-05 06:04:49,671 WARNING Found /var/run/reboot-required, rebooting
2024-09-05 06:04:49,689 WARNING Shutdown msg: b"Shutdown scheduled for Fri 2024-09-06 06:00:00 UTC, use 'shutdown -c' to cancel."
```

Mira la hora: 06:04:49 en UTC. O sea que sí se ejecutó a la 1am. Pero no se reinició?

No lo hizo. La reiniciada quedó programada para el día Viernes 6 de Septiembre:
```
WARNING Shutdown msg: b"Shutdown scheduled for Fri 2024-09-06 06:00:00 UTC
```

O sea que mañana (hoy es Jueves, 5 de Septiembre), deberá reiniciarse el servidor.

En esta [pregunta](https://askubuntu.com/questions/934807/unattended-upgrades-status) hay más detalles.

## Servidor reinició exitosamente

Así como decía el mensaje que estaba programada la reiniciada, así pasó y Nginx y Sidekiq quedaron corriendo sin problemas.

### Revisión de Nginx luego de reinicio

Me llamó la atención que no aparecían el proceso de Cash Flow:
```
ubuntu@localhost:~$ sudo service nginx status
[sudo] password for ubuntu: 
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-09-06 06:01:25 UTC; 8h ago
       Docs: man:nginx(8)
   Main PID: 694 (nginx)
      Tasks: 31 (limit: 1124)
     Memory: 234.0M
     CGroup: /system.slice/nginx.service
             ├─  629 Passenger watchdog
             ├─  671 Passenger core
             ├─  694 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─  701 nginx: worker process
             └─10968 Passenger RubyApp: /home/ubuntu/coshinotes/app (production)

Sep 06 06:01:24 localhost systemd[1]: Starting A high performance web server and a reverse proxy server...
Sep 06 06:01:25 localhost systemd[1]: Started A high performance web server and a reverse proxy server.
```

Sin embargo, al ir al sitio web y revisar de nuevo, sí aparecieron. Como que estaban dormidos:
```
ubuntu@localhost:~$ sudo service nginx status
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-09-06 14:26:40 UTC; 27s ago
       Docs: man:nginx(8)
    Process: 17914 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 17915 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 17930 (nginx)
      Tasks: 53 (limit: 1124)
     Memory: 436.8M
     CGroup: /system.slice/nginx.service
             ├─17916 Passenger watchdog
             ├─17920 Passenger core
             ├─17930 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─17936 nginx: worker process
             ├─17945 Passenger AppPreloader: /home/ubuntu/cashflow/app
             ├─17996 Passenger RubyApp: /home/ubuntu/cashflow/app (production)
             ├─18018 Passenger AppPreloader: /home/ubuntu/coshinotes/app
             ├─18073 Passenger RubyApp: /home/ubuntu/coshinotes/app (production)
             └─18097 Passenger RubyApp: /home/ubuntu/coshinotes/app (production)
```


### Revisión de Sidekiq luego de reinicio

Sidekiq parecía estar todo en orden. Supongo porque lo tengo montado con Systemd.
```
ubuntu@localhost:~$ systemctl --user status sidekiq.service
● sidekiq.service - sidekiq
     Loaded: loaded (/home/ubuntu/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-09-06 14:24:44 UTC; 1min 12s ago
   Main PID: 17432 (bundle)
     CGroup: /user.slice/user-1000.slice/user@1000.service/sidekiq.service
             └─17432 sidekiq 6.5.12 app [0 of 1 busy]

Sep 06 14:24:44 localhost systemd[17425]: Started sidekiq.
Sep 06 14:24:47 localhost sidekiq[17432]: 2024-09-06T14:24:47.720Z pid=17432 tid=19g INFO: Booting Sidekiq 6.5.12 with Sidekiq::RedisConnection::RedisAdapter options {:url=>nil}
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.934Z pid=17432 tid=19g INFO: Booted Rails 7.0.4 application in production environment
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.935Z pid=17432 tid=19g INFO: Running in ruby 3.1.0p0 (2021-12-25 revision fb4df44d16) [x86_64-linux]
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.935Z pid=17432 tid=19g INFO: See LICENSE and the LGPL-3.0 for licensing details.
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.935Z pid=17432 tid=19g INFO: Upgrade to Sidekiq Pro for more features and support: https://sidekiq.org
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.937Z pid=17432 tid=19g INFO: Loading Schedule
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.937Z pid=17432 tid=19g INFO: Scheduling recurring_expenditures {"class"=>"RecurringExpendituresWorker", "cron"=>"0 0 1 * *", "queue"=>"default"}
Sep 06 14:24:48 localhost sidekiq[17432]: 2024-09-06T14:24:48.946Z pid=17432 tid=19g INFO: Schedules Loaded
```

### Uptime

Finalmente, el uptime cambió a apenas un par de horas:
```
ubuntu@localhost:~$ uptime
 14:31:23 up  8:30,  1 user,  load average: 0.00, 0.01, 0.00
```

### Revisión de Mega CMD luego de reinicio


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

## Estado de Mega CMD