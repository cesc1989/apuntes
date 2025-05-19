# Configurando Entorno de Desarrollo - Parte 1

# Instalando y desinstalando postgresql en Macos Ventura con Homebrew

## Desintalar

Para desinstalar, ejemplo, postgresql@14:

```bash
$ brew uninstall --ignore-dependencies postgresql@14

Uninstalling /opt/homebrew/Cellar/postgresql@14/14.13... (3,323 files, 45.5MB)
```

O también así. En este caso postgresql 16
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

