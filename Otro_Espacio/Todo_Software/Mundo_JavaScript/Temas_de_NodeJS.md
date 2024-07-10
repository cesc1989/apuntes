# Temas de NodeJS

Artículos enfocados a esta plataforma que permite ejecutar JavaScript de lado del servidor y revolucionó la forma de escribir código con este lenguaje en el backend y en el frontend.

# Troubleshooting in NodeJS

## node-sass missing binding error

Message:

    Error: Missing binding /app/node_modules/node-sass/vendor/linux-x64-48/binding.node
    Node Sass could not find a binding for your current environment: Linux 64-bit with Node.js 6.x
    
    Found bindings for the following environments:
      - OS X 64-bit with Node.js 6.x
    
    This usually happens because your environment has changed since running `npm install`.
    
    Run `npm rebuild node-sass` to build the binding for your current environment.

Can be solved by running:

    npm rebuild node-sass

or

    rm rf node_modules
    npm install

Ver [este issue](https://github.com/sass/node-sass/issues/1585) para entender mejor.

## cannot run in wd

    npm WARN lifecycle askYeoman@0.0.0~postinstall: cannot run in wd %s %s (wd=%s) askYeoman@0.0.0 bower install /app

Solución: `bower install --allow-root`

Ver [Stack Overflow](https://stackoverflow.com/questions/18136746/npm-install-failed-with-cannot-run-in-wd#19132229).

# NPM

## About `**npm install**` locally and/or globally

There are two ways to install npm packages: locally or globally. Choose which kind of installation to use based on how you want to use the package.

- If you want to depend on the package from your own module, using something like Node.js' require, then you want to install locally. This is npm install's default behavior.
- If you want to use a package as a command line tool, (such as grunt CLI), then [install it globally](https://docs.npmjs.com/getting-started/installing-npm-packages-globally).

## Cómo corregir conflictos del `package-lock.json`

Una forma es:

    rm package-lock.json
    npm install
    git add package-lock.json
    git commit

pero no es ideal. Vista en [Blog de Adriaan](https://blog.adriaan.io/merge-conflict-in-package-lock-json.html). La otra forma es usando `npm-merge-driver`, el cual le enseña a git cómo corregir tales conflictos. 

Se puede instalar `$ npx npm-merge-driver install -g` y corrige los conflictos al hacer git merge.

Visto en [SO](https://stackoverflow.com/questions/50160311/auto-merging-package-lock-json).

## Otros Temas

- About `package.json`: [docs](https://docs.npmjs.com/files/package.json#dependencies)
- `npm install` usando el `package-lock.json`: [SO](https://stackoverflow.com/questions/45022048/why-does-npm-install-rewrite-package-lock-json)

# Instalando Node 14 en Macos M1

Resulta que necesitaba instalar esta vaina pero no se puede. Según el [Readme de NVM](https://github.com/nvm-sh/nvm?tab=readme-ov-file#troubleshooting-on-macos):

> **Note** For Macs with the Apple Silicon chip, node started offering **arm64** arch Darwin packages since v16.0.0 and experimental **arm64** support when compiling from source since v14.17.0. If you are facing issues installing node using `nvm`, you may want to update to one of those versions or later.

Cuando intento, me deja este error:

```
$ nvm install 14.17.0
Downloading and installing node v14.17.0...
Downloading https://nodejs.org/dist/v14.17.0/node-v14.17.0-darwin-arm64.tar.xz...
curl: (22) The requested URL returned error: 404

Binary download from https://nodejs.org/dist/v14.17.0/node-v14.17.0-darwin-arm64.tar.xz failed, trying source.
grep: /Users/francisco/.nvm/.cache/bin/node-v14.17.0-darwin-arm64/node-v14.17.0-darwin-arm64.tar.xz: No such file or directory
Provided file to checksum does not exist.
Binary download failed, trying source.
Clang v3.5+ detected! CC or CXX not specified, will use Clang as C/C++ compiler!
Local cache found: ${NVM_DIR}/.cache/src/node-v14.17.0/node-v14.17.0.tar.xz
Checksums match! Using existing downloaded archive ${NVM_DIR}/.cache/src/node-v14.17.0/node-v14.17.0.tar.xz
$>./configure --prefix=/Users/francisco/.nvm/versions/node/v14.17.0 <
pyenv: python3.9: command not found

The `python3.9' command exists in these Python versions:
  3.9.7

Note: See 'pyenv help global' for tips on allowing both
      python2 and python3 to be found.
nvm: install v14.17.0 failed!
```

El cual es lo mismo que se ve en [este issue](https://github.com/nvm-sh/nvm/issues/3026#issuecomment-1419726966) donde también mencionan lo de no poder instalar versión 14.

## Solución

Al final pude instalar haciendo un cambio de arquitectura desde la terminal para que NVM pudiera hacer la instalación. Se hace así:

```bash
arch -x86_64 bash # para cambiar de arm64 a i386

# así verifiqué
arch
i386
```

Luego pude instalar node v14.17 como si nada:
```bash
nvm install 14.17.0
Downloading and installing node v14.17.0...
Downloading https://nodejs.org/dist/v14.17.0/node-v14.17.0-darwin-x64.tar.xz...
######################################################################################################################################################################################################################################################### 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v14.17.0 (npm v6.14.13)
```

Cuando verificaba la versión instalada, así se reflejaba:
```
node -v
v14.17.0
```

Finalmente, para cambiar de arquitectura ejecuto el 1er comando pero indicando la Apple Sillicon
```bash
arch -arm64 bash
```

Visto en:
- [Stack Overflow](https://stackoverflow.com/a/67907214/1407371)
- [Devzilla](https://devzilla.io/using-nodejs-14-with-mac-silicon-m1)