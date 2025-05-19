# Configurando Entorno de Desarrollo - Parte 1

# Instalando y desinstalando postgresql en Macos Ventura con Homebrew

## Desintalar

Para desinstalar postgresql@14:

```bash
$ brew uninstall --ignore-dependencies postgresql@14

Uninstalling /opt/homebrew/Cellar/postgresql@14/14.13... (3,323 files, 45.5MB)
```

O postgresql 16:
```bash
brew uninstall --force postgresql@16

rm -rf /opt/homebrew/var/postgres
```

## Instalar

Para instalar postgresql@16:
```bash
brew install postgresql@16
```

Servicios de postgres con Brew. Aquí muestra cuando estaban las dos versiones instaladas
```bash
$ brew services 
Name          Status  User      File
postgresql@14 started francisco ~/Library/LaunchAgents/homebrew.mxcl.postgresql@14.plist
postgresql@16 none              
redis         started francisco ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
unbound       none 
```

Apagando uno:
```
$ brew services stop postgresql@14
Stopping `postgresql@14`... (might take a while)
==> Successfully stopped `postgresql@14` (label: homebrew.mxcl.postgresql@14)
```

Iniciando el otro:
```bash
$ brew services start postgresql@16
==> Successfully started `postgresql@16` (label: homebrew.mxcl.postgresql@16)
```

Creando los enlaces simbolicos:
```bash
$ brew unlink postgresql@16 && brew link --force postgresql@16
Unlinking /opt/homebrew/Cellar/postgresql@16/16.4... 1683 symlinks removed.
Linking /opt/homebrew/Cellar/postgresql@16/16.4... 1683 symlinks created.
```

# Instalar PostgreSQL 16/17 en Linux Mint

En este [tutorial dicen](https://medium.com/@mglaving/how-to-install-postgresql-16-on-linux-mint-21-d58e875fe7c6) que instalan psql 16 con esos comandos pero en el LG Gram termino instalando la version 17.

Comandos:
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

sudo apt update && sudo apt upgrade -y

sudo apt install postgresql -y
```

Ya vi el error. Para especificar una version hay que agreagarla a la instruccion de instalacion:

> If you want a specific version, use 'postgresql-15' or similar instead of 'postgresql'

## Configuracion de psql

Para poder abrir `psql` con mi usuario de sistema hay que primero crear el rol como el usuario postgres. Sigue estos comandos:

```
sudo su postgres -c psql

CREATE ROLE cesc SUPERUSER;

ALTER ROLE cesc WITH LOGIN;
```

Para mas ver [[Otro_Espacio/CheatSheets/postgresql/queries/README|README]]

# Instalar libvips en Linux Mint

Bullet Train sugiere este comando:
```
sudo apt-get install libvips
```

Y al parecer instalo todo pero el comando `bin dev/setup` de Bullet Train decia que no lo encontraba en el PATH.

Hay que instalarlo con la libreria version dev:
```
sudo apt-get install libvips libvips-dev
```

# Actualizar postgresql@16 a versión 17

Las claves son:

- Desactivar extensiones molestas como Hypopg o Postgis.

## Instalar postgresql@17

Se instala así:
```
HOMEBREW_NO_AUTO_UPDATE=1 brew install postgresql@17
```

## Copia datos de postgresql@16 a postgresql@17

Apaga primero la versión anterior:
```
brew services stop postgresql@16
```

Crea carpetas de sockets para esta operación:
```
mkdir -p /tmp/pg_upgrade_sockets
chmod 700 /tmp/pg_upgrade_sockets
```

Ejecuta el comando `pg_upgrade`:
```
LC_ALL=en_US.UTF-8 \
/opt/homebrew/opt/postgresql@17/bin/pg_upgrade \
  -b /opt/homebrew/opt/postgresql@16/bin \
  -B /opt/homebrew/opt/postgresql@17/bin \
  -d /opt/homebrew/var/postgresql@16 \
  -D /opt/homebrew/var/postgresql@17 \
  -o "-c unix_socket_directories='/tmp/pg_upgrade_sockets'" \
  -O "-c unix_socket_directories='/tmp/pg_upgrade_sockets'"
```

### Elimina una BD molesta

Había un problema con `grimoire_dev` por PostGIS así que decidí borrarla:

```
/opt/homebrew/opt/postgresql@16/bin/psql -U francisco -d postgres -c "DROP DATABASE grimoire_dev;"
```

### Problemas con Hypopg

Las bases de datos de Edge usan esta extensión. Estaba molestando porque la instalación no es tan sencilla. Cuando instalaba con brew se iba a postgresql@14 y cuando lo hacía por fuentes se iba a la versión 16.

Decidí desinstalar esa extensión:
```
/opt/homebrew/opt/postgresql@16/bin/psql -U francisco -d francisco -c "DROP EXTENSION IF EXISTS hypopg;"
/opt/homebrew/opt/postgresql@16/bin/psql -U francisco -d luna_api_development_replica -c "DROP EXTENSION IF EXISTS hypopg;"
/opt/homebrew/opt/postgresql@16/bin/psql -U francisco -d luna_api_development_3 -c "DROP EXTENSION IF EXISTS hypopg;"
/opt/homebrew/opt/postgresql@16/bin/psql -U francisco -d luna_api_development_4 -c "DROP EXTENSION IF EXISTS hypopg;"
/opt/homebrew/opt/postgresql@16/bin/psql -U francisco -d luna_api_test -c "DROP EXTENSION IF EXISTS hypopg;"
```

## Iniciar postgresql@17

Finalmente pude completar la migración y levantar la nueva versión de PostgreSQL:

```
brew services start postgresql@17
```

Se puede verificar que corra con estos otros comando:
```
brew services list
Name          Status  User      File
postgresql@16 none
postgresql@17 started francisco ~/Library/LaunchAgents/homebrew.mxcl.postgresql@17.plist
redis         started francisco ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
unbound       none
```

O con este `pg_isready`:
```
pg_isready
/tmp:5432 - accepting connections
```