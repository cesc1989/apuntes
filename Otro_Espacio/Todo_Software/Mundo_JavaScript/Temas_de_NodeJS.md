# Temas de NodeJS
Artículos enfocados a esta plataforma que permite ejecutar JavaScript de lado del servidor y revolucionó la forma de escribir código con este lenguaje en el backend y en el frontend.

## Troubleshooting in NodeJS

**node-sass missing binding error**
Message:

    Error: Missing binding /app/node_modules/node-sass/vendor/linux-x64-48/binding.node
    Node Sass could not find a binding for your current environment: Linux 64-bit with Node.js 6.x
    
    Found bindings for the following environments:
      - OS X 64-bit with Node.js 6.x
    
    This usually happens because your environment has changed since running `npm install`.
    
    Run `npm rebuild node-sass` to build the binding for your current environment.

Can be solved by issuing:

    npm rebuild node-sass

or

    rm rf node_modules
    npm install

Ver [este issue](https://github.com/sass/node-sass/issues/1585) para entender mejor.

**cannot run in wd**

    npm WARN lifecycle askYeoman@0.0.0~postinstall: cannot run in wd %s %s (wd=%s) askYeoman@0.0.0 bower install /app

Solución: `bower install --allow-root`

Ver [Stack Overflow](https://stackoverflow.com/questions/18136746/npm-install-failed-with-cannot-run-in-wd#19132229).

## NPM

**About** `**npm install**` **locally and/or globally**
There are two ways to install npm packages: locally or globally. Choose which kind of installation to use based on how you want to use the package.

- If you want to depend on the package from your own module, using something like Node.js' require, then you want to install locally. This is npm install's default behavior.
- If you want to use a package as a command line tool, (such as grunt CLI), then [install it globally](https://docs.npmjs.com/getting-started/installing-npm-packages-globally).

**Cómo corregir conflictos del** `**package-lock.json**`
Una forma es:


    rm package-lock.json
    npm install
    git add package-lock.json
    git commit

pero no es ideal. Vista en [Blog de Adriaan](https://blog.adriaan.io/merge-conflict-in-package-lock-json.html). La otra forma es usando `npm-merge-driver`, el cual le enseña a git cómo corregir tales conflictos. 

Se puede instalar `$ npx npm-merge-driver install -g` y corrige los conflictos al hacer git merge

Visto en [SO](https://stackoverflow.com/questions/50160311/auto-merging-package-lock-json).

**Otros Temas**

- About `package.json`: [docs](https://docs.npmjs.com/files/package.json#dependencies)
- `npm install` usando el `package-lock.json`: [SO](https://stackoverflow.com/questions/45022048/why-does-npm-install-rewrite-package-lock-json)

