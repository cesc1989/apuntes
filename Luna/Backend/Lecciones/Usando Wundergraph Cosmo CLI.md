# Instalación de WunderGraph Cosmo CLI

Se instala así:
```bash
npm install -g wgc@latest
```

La versión de Luna es 0.75.3:
```bash
npm install -g wgc@0.75.3
```

## Comandos

En macos, para crear el graph a revisar se genera con:
```bash
bundle exec rails luna_api:federation:dump_sdl \
&& sed -i '' 's/federation__//g' luna_api_federation_sdl.graphql \
```

Para revisar el graph:
```bash
wgc subgraph check backend --schema ./luna_api_federation_sdl.graphql
```

## Usar NVM default

Para crear una versión por default:
```bash
nvm alias default 22.14.0
```

Para usarla:
```bash
nvm use default
```

## Error de permisos al instalar

> [!Note]
> Esto funciona pero ojo. Es porque no estaba usando la versión de Node de usuario sino la de root.
>
> Todo lo que se hizo a continuación está mal.

Estuve intentando instalar con el comando sugerido por los docs pero me daba error de permisos???
```bash
npm ERR! code EACCES
npm ERR! syscall symlink
npm ERR! path ../lib/node_modules/wgc/dist/src/index.js
npm ERR! dest /usr/local/bin/wgc
npm ERR! errno -13
npm ERR! Error: EACCES: permission denied, symlink '../lib/node_modules/wgc/dist/src/index.js' -> '/usr/local/bin/wgc'
npm ERR!  [Error: EACCES: permission denied, symlink '../lib/node_modules/wgc/dist/src/index.js' -> '/usr/local/bin/wgc'] {
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'symlink',
npm ERR!   path: '../lib/node_modules/wgc/dist/src/index.js',
npm ERR!   dest: '/usr/local/bin/wgc'
npm ERR! }
npm ERR!
npm ERR! The operation was rejected by your operating system.
npm ERR! It is likely you do not have the permissions to access this file as the current user
npm ERR!
npm ERR! If you believe this might be a permissions issue, please double-check the
npm ERR! permissions of the file and its containing directories, or try running
npm ERR! the command again as root/Administrator.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/francisco/.npm/_logs/2025-04-14T14_25_19_109Z-debug-0.log
```

Primero probé solo cambiar el dueño de la carpeta `node_modules` pero no sirvió:
```bash
sudo chown -R francisco /usr/local/lib/node_modules
```

Lo que sí sirvió fue este comando:
```bash
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
```

¿Qué hace?

Primero este `npm config get prefix`:
```bash
$ npm config get prefix
/usr/local
```

Esa es la carpeta donde está todo lo de node.

Luego el chown combinado lo que hace es cambiar el dueño a `francisco` en las carpetas:

- `/usr/local/lib/node_modules`
- `/usr/local/bin`
- `/usr/local/share`

```bash
$ ls -l /usr/local/
total 0
drwxr-xr-x  16 root       wheel  512 Jul  3  2024 aws-cli
drwxr-xr-x  31 francisco  wheel  992 Apr 14 10:03 bin
# (...)
drwxr-xr-x   5 francisco  wheel  160 Feb 17  2023 share

$ ls -l /usr/local/lib/
total 97656
drwxr-xr-x  3 root       wheel        96 Feb 21  2023 docker
drwxr-xr-x  3 root       wheel        96 Jun  1  2022 dtrace
# (...)
drwxr-xr-x  5 francisco  wheel       160 Apr 14 09:25 node_modules
```

Visto en [Stack Overflow](https://stackoverflow.com/a/66301922/1407371).