# Migración WordPress de Hosting a AWS

## AWS
- Amazon no permite pings a sus servidores: [SuperUser](http://superuser.com/questions/1114691/ping-amazon-com-gives-100-packet-loss-on-aws-ec2-instance#1114693)
- RDS Storage: [magnetico, ssd, iops](http://jayendrapatil.com/aws-rds-storage/)
- Conectarse a EC2 via SFTP: [SFTP filezilla](http://stackoverflow.com/questions/16744863/connect-to-amazon-ec2-file-directory-using-filezilla-and-sftp)
- Cómo hacer upgrade/downgrade de instancias de EC2: [Will Warren](https://willwarren.com/2014/07/15/resize-ec2-instances-minimal-downtime/)


## MySQL
- Mysql Workbench error 1142 “Error querying security information” on Data Export: [solución en ubuntu 14.04 es desinstalar workbench 6.3.6 e instalar 6.3.4](http://stackoverflow.com/questions/34521822/mysql-workbench-error-1142-error-querying-security-information-on-data-export)
- Conectarse a MySQL Server instalado en Vagrant: [cambiar el](http://stackoverflow.com/questions/10709334/how-to-connect-to-mysql-server-inside-virtualbox-vagrant#10794530) `[bind-address](http://stackoverflow.com/questions/10709334/how-to-connect-to-mysql-server-inside-virtualbox-vagrant#10794530)` y mediante [workbench](http://stackoverflow.com/a/29604714/1407371)
- Hacer backup desde consola, compreso y restaurar: [Web](http://99webtools.com/blog/backup-and-restore-large-mysql-database/)
- Backup de tablas en especifico: [DBA Exchange](https://dba.stackexchange.com/questions/9306/how-do-you-mysqldump-specific-tables)
- extraer info de archivo de backup: [Blog](http://gtowey.blogspot.com.co/2009/11/restore-single-table-from-mysqldump.html)
- recuperar tablas en especifico: [SO](http://stackoverflow.com/a/16214902/1407371)

**Hacer MySQL backup desde consola**


    mysqldump -u \[uname] -p [pass\] [dbname] > [backupfile.sql]

Que sea compreso:

    sqldump -u \[uname] -p[pass\] [dbname] | gzip -9 > [backupfile.sql.gz]

Restaurar backup desde consola

    mysql -u \[uname] -p[pass\] [db_to_restore] < [backupfile.sql]

Restaurar backup compreso:

    gunzip < \[backupfile.sql.gz] | mysql -u [uname] -p[pass\] [dbname]


## Web Server
- Cómo saber qué web server corre un sitio web: solución [usando](https://www.quora.com/How-can-I-determine-which-web-server-a-particular-website-is-using-Apache-IIS-Nginx-etc) `[curl -I URL](https://www.quora.com/How-can-I-determine-which-web-server-a-particular-website-is-using-Apache-IIS-Nginx-etc)` [o con el inspector de chrome](https://www.quora.com/How-can-I-determine-which-web-server-a-particular-website-is-using-Apache-IIS-Nginx-etc)
- Apache Traffic Server o [ATS](http://trafficserver.apache.org/)
- `dig A` para obtener la IP de un servicio externo y agregarla a los security groups y permitirle peticiones entrantes
- [mantener el query string en nginx](http://serverfault.com/questions/685525/nginx-php-fpm-query-parameters-wont-be-passed-to-php)
- [apache2 ocupa el puerto de nginx](http://stackoverflow.com/questions/14972792/nginx-nginx-emerg-bind-to-80-failed-98-address-already-in-use#15101745)
- URLs limpias en Nginx: [SO](http://stackoverflow.com/questions/8967682/nginx-rewriting-rule-for-getting-clean-url#8968267)
- CNAME para dominio raíz: [ANAME](http://stackoverflow.com/questions/16022324/how-to-setup-dns-for-an-apex-domain-no-www-pointing-to-a-heroku-app)
- Redirect from www subdomain to apex domain and viceversa: [The Site Wizard](https://www.thesitewizard.com/apache/redirect-domain-www-subdomain.shtml)
- Retornar 301 desde dominio raíz a subdominio www: [Server Fault](https://serverfault.com/questions/641266/forward-the-root-domain-to-the-www-subdomain-using-dns-records)
- nginx variables: [Server Fault](https://serverfault.com/questions/706438/what-is-the-difference-between-nginx-variables-host-http-host-and-server-na#706439)
- [H5BP Nginx Server Configs](https://github.com/h5bp/server-configs-nginx)


## PHP
- Qué es PHP-FPM: [SO](http://stackoverflow.com/questions/4526242/what-is-the-difference-between-fastcgi-and-fpm)
- [PHP Brew](https://phpbrew.github.io/phpbrew/) [AskUbuntu](http://askubuntu.com/questions/550191/install-php-5-4-on-ubuntu-14-04-lts-without-compiling)
- PHP [stripslashes](https://codex.wordpress.org/Function_Reference/stripslashes_deep), [magic_quotes removed from php 5.4](http://us3.php.net/manual/en/info.configuration.php#ini.magic-quotes-gpc)
- Problemas en carga de archivos: [php.ini y nginx.conf](https://atulhost.com/fix-nginx-error-413-request-entity-too-large)
- [Entity too large](https://wordpress.org/support/topic/413-request-entity-too-large-9/)
- `Unknow instance` al recargar php-fpm luego de haber cambiado el pool: [SO](http://stackoverflow.com/a/29253579/1407371) - [Github, no la solución pero mecanismos de](https://github.com/oerdnj/deb.sury.org/issues/341) `[debugging](https://github.com/oerdnj/deb.sury.org/issues/341)`
- [PHP FPM example conf](https://github.com/perusio/php-fpm-example-config)

**Cambiar el tamaño de archivos para subir y el peso del post data**

- [php.ini](http://stackoverflow.com/questions/2184513/php-change-the-maximum-upload-file-size#2184541)

PHP.ini recommended WordPress configuration:

- `max_execution_time` 60
- `memory_limit` 128M
- `post_max_size` 48M
- `upload_max_filesize` 48M

**PHP-FPM performance issues**

- [Laracastas](https://laracasts.com/discuss/channels/servers/php-fpm-is-using-100-cpu)
- [SO](http://stackoverflow.com/questions/13732436/php5-fpm-randomly-starts-consuming-a-lot-of-cpu)

**Restart PHP services command**

PHP5 service restart command

    sudo service php5-fpm restart

PHP7 service restart command

    sudo service php7.0-fpm restart


## PHP7
- Instalar PHP 7 en Ubuntu 14.04: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-upgrade-to-php-7-on-ubuntu-14-04) *ojo con la ruta del sock de php7.0-fpm*

**PHP-7.1 módulos recomendados**

    sudo apt-get install -y php7.1-fpm php7.1-cli php7.1-curl php7.1-mysql php7.1-sqlite3 \
        php7.1-gd php7.1-xml php7.1-mcrypt php7.1-mbstring php7.1-iconv


## Varnish
- Configurar Varnish con Nginx: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-nginx-php-and-varnish-on-ubuntu-12-04) [Linode](https://www.linode.com/docs/websites/varnish/getting-started-with-varnish-cache/)
- Configurar Varnish con Nginx y SSL: [Linode](https://www.linode.com/docs/websites/varnish/use-varnish-and-nginx-to-serve-wordpress-over-ssl-and-http-on-debian-8)

**Varnish setup with SSL**
Over HTTP
www -> varnish on port 80 -> redirects to nginx on port 8080

Over HTTPS
www -> varnish on port 443 -> redirects to nginx on port 8080


## WordPress
- Extensiones comunes para PHP: [WordPress Development](https://wordpress.stackexchange.com/questions/42098/what-are-php-extensions-and-libraries-wp-needs-and-or-uses)
- **Permisos en WordPress** para editar archivos desde el admin y carga de plugins: [cambiar usuario y grupo de sistema a la carpeta que contiene sitio](http://stackoverflow.com/questions/17768201/wordpress-on-ec2-requires-ftp-credentials-to-install-plugins) - [también en Smashing Magazine](https://www.smashingmagazine.com/2014/05/proper-wordpress-filesystem-permissions-ownerships/#permissions-for-a-standard-wordpress-server-configuration)
- Cambiar clave de usuario wordpress: [MD5 online](http://www.wpbeginner.com/beginners-guide/how-to-reset-a-wordpress-password-from-phpmyadmin/)
- Permisos wordpress: [Codex](https://codex.wordpress.org/Changing_File_Permissions) | [uploads folder](http://stackoverflow.com/a/32144990)
- WordPress Site URLs: [WordPress Codex](https://codex.wordpress.org/Changing_The_Site_URL) *las que están guardadas en la base de datos*
- [WordPress Permissions Update Error](https://aaronjholbrook.com/wordpress-permissions-update-error-resolved/) `[the update cannot be installed because we will be unable](https://aaronjholbrook.com/wordpress-permissions-update-error-resolved/)`

`**wp_options**` **table**

- [Understanding and Working with The WordPress Options Table](https://code.tutsplus.com/tutorials/understanding-and-working-with-the-wordpress-options-table--cms-21119).
- [Option Reference](https://codex.wordpress.org/Option_Reference)
- [Codex Reference](https://codex.wordpress.org/Database_Description#Table:_wp_options).

**New Relic + WordPress**

- [PHP7 support in NewRelic](https://discuss.newrelic.com/t/php-7-support-is-out/35274/3)
- [New Relic + WordPress](https://code.tutsplus.com/tutorials/using-new-relic-to-monitor-wordpress-performance--cms-22002)
- [Optimize WordPress](https://blog.newrelic.com/2013/02/07/web-performance-optimization-automation/)


## Linux
- Remove host key de `known_hosts`: [Ask Ubuntu](https://askubuntu.com/questions/20865/is-it-possible-to-remove-a-particular-host-key-from-sshs-known-hosts-file)
- **Cómo agregar usuario a grupo de sistema**: [ask ubuntu](http://askubuntu.com/questions/79565/add-user-to-existing-group)
- Ver los grupos a los que un usuario pertenece: [SO](http://stackoverflow.com/questions/350141/how-to-find-out-what-group-a-given-user-has)
- Problema SSH debug1: expecting SSH2_MSG_KEX_DH_GEX_GROUP: solución [modificar el archivo](https://superuser.com/questions/568891/ssh-works-in-putty-but-not-terminal) `[/etc/ssh/sshd_config](https://superuser.com/questions/568891/ssh-works-in-putty-but-not-terminal)` [local](https://superuser.com/questions/568891/ssh-works-in-putty-but-not-terminal)
- `nohup` para correr comando en segundo plano: [Unix Exchange](http://unix.stackexchange.com/questions/65116/does-a-scp-transfer-close-when-i-close-the-shell)
- Encontrar paquete para instalar que se instaló con el .deb [Esto](http://stackoverflow.com/questions/9989483/how-to-remove-mysql-workbench-5-2-in-ubuntu-unity#9992953) o [Esto otro](http://stackoverflow.com/a/26951005/1407371)
- Cómo saber qué versión de Linux estoy usando desde consola: [solución](http://www.howtogeek.com/206240/how-to-tell-what-distro-and-version-of-linux-you-are-running/)
- Usar rsync con ssh y archivo de identidad: [SO](http://stackoverflow.com/questions/5527068/how-do-you-use-an-identity-file-with-rsync#5527157) y [How to geek](http://www.howtogeek.com/135533/how-to-use-rsync-to-backup-your-data-on-linux/)
- deshacer un `rm` en linux: [Super User](https://superuser.com/questions/32355/undo-linuxs-rm)


## Local Development
- VirtualBox no permite redirección de puertos menores al 1024: [SO](http://stackoverflow.com/questions/17437137/vagrant-wont-forward-only-port-80#17437627)
- Cambiar folder principal compartido de `/vagrant` a `/home/vagrant` causa que se pida clava al hacer `vagrant ssh` [SO](http://stackoverflow.com/questions/23599297/changing-vagrantfile-causes-vagrant-ssh-to-prompt-for-a-password#23775432)


## Let’s Encrypt
- Let'sencrypt `Too many invalid authorizations recently`: [Letsencrypt discourse 1](https://community.letsencrypt.org/t/letsencrypt-win-simple-too-many-authorisations-hitting-rate-limit/32395) - [Letsencrypt discourse 2](https://community.letsencrypt.org/t/how-long-i-got-block-from-invalid-authorization/31584/4) - [Rate Limits](https://letsencrypt.org/docs/rate-limits/) - [Staging env for letsencrypt](https://letsencrypt.org/docs/staging-environment/)
- Let's encrypt `[To fix these errors, please make sure that your domain name was entered correctly and the DNS A record(s) for that domain contain(s) the right IP address](https://community.letsencrypt.org/t/to-fix-these-errors-please-make-sure-that-your-domain-name-was-entered-correctly-and-the-dns-a-record-s-for-that-domain-contain-s-the-right-ip-address/25661)` - Revisar ruta del sitio en `/etc/letsencrypt/renewal/[PROJECT-NAME].conf`

