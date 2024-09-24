# Configurando Entorno de Desarrollo - Parte 1

# Instalando y desinstalando postgresql en Macos Ventura con Homebrew

Para desinstalar, ejemplo, postgresql@14:

```bash
$ brew uninstall --ignore-dependencies postgresql@14

Uninstalling /opt/homebrew/Cellar/postgresql@14/14.13... (3,323 files, 45.5MB)
```

O también así. En este caso postgresql¡6
```bash
brew uninstall --force postgresql@16

rm -rf /opt/homebrew/var/postgres
```

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