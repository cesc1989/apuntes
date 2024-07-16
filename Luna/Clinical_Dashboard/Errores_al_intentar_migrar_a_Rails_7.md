# Errores al intentar migrar a Rails 7

Cómo hacer la migración → [[Migración_a_Rails_7]]

## DEPRECATION WARNING: Using legacy connection handling is deprecated. Please set `legacy_connection_handling` to `false` in your application

Se arregla configurando dicho setting en el archivo del entorno tal como se [menciona en la guía](https://guides.rubyonrails.org/active_record_multiple_databases.html#migrate-to-the-new-connection-handling).


## ActionView::Template::Error: Webpacker can't find common in

Complete error in tests:

    Failure/Error: <%= javascript_pack_tag 'common' %>
         
         ActionView::Template::Error:
           Webpacker can't find common in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/public/packs-test/manifest.json. Possible causes:
           1. You want to set webpacker.yml value of compile to true for your environment
              unless you are using the `webpack -w` or the webpack-dev-server.
           2. webpack has not yet re-run to reflect updates.
           3. You have misconfigured Webpacker's config/webpacker.yml file.
           4. Your webpack configuration is not creating a manifest.
           Your manifest contains:
           {
             "entrypoints": {
             }
           }

En este [issue se menciona](https://github.com/rails/webpacker/issues/1047#issuecomment-347674037) que la posible solución está al correr estos comandos:

    NODE_ENV=test bundle exec rails webpacker:compile
    
    RAILS_ENV=test bundle exec rails webpacker:compile

Eso sí se hace para el CI pero cuando lo hago en local no funciona. Cuando lo ejecuto en local obtengo el siguiente error.


## Entry module not found: Error: Can't resolve 'babel-loader'

El paquete `babel-loader` no lo tengo en el `package.json` principal porque se supone que ya al incluir `@rails/webpacker` debería quedar instalado. Y así es!


    $ ls -l node_modules/@rails/webpacker/node_modules
    total 0
    drwxr-xr-x  83 fquintero  staff  2656 Sep  8 10:25 @babel
    drwxr-xr-x   6 fquintero  staff   192 Sep  8 10:25 ansi-styles
    drwxr-xr-x   8 fquintero  staff   256 Sep  8 10:25 babel-loader
    ...

¿Por qué no lo encuentra?

Error completo:

    EntryModuleNotFoundError: Entry module not found: Error: Can't resolve 'babel-loader' in '/Users/fquintero/projects/luna-project/clinical-dashboard-backend'
    
    
    resolve 'babel-loader' in '/Users/fquintero/projects/luna-project/clinical-dashboard-backend'
      Parsed request is a module
      using description file: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/package.json (relative path: .)
        resolve as module
          /Users/fquintero/projects/luna-project/node_modules doesn't exist or is not a directory
          /Users/fquintero/projects/node_modules doesn't exist or is not a directory
          /Users/fquintero/node_modules doesn't exist or is not a directory
          /Users/node_modules doesn't exist or is not a directory
          /node_modules doesn't exist or is not a directory
          looking for modules in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules
            using description file: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/package.json (relative path: ./node_modules)
              using description file: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/package.json (relative path: ./node_modules/babel-loader)
                no extension
                  /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader doesn't exist
                .js
                  /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader.js doesn't exist
                .json
                  /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader.json doesn't exist
                as directory
                  /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader doesn't exist

Intentando solo compilar con webpack:

    $ ./bin/webpack
    
    Insufficient number of arguments or no entry found.
    Alternatively, run 'webpack(-cli) --help' for usage info.
    
    Hash: a178e4e16c6c3a9da042
    Version: webpack 4.46.0
    Time: 54ms
    Built at: 09/09/2022 9:31:41 AM
            Asset      Size  Chunks             Chunk Names
    manifest.json  23 bytes          [emitted]  
    
    ERROR in Entry module not found: Error: Can't resolve 'babel-loader' in '/Users/fquintero/projects/luna-project/clinical-dashboard-backend'

Cuando procedo a instalar `babel-loader` da error con otro plugin que está configurado en el archivo `babel.config.js`:

    $ yarn add babel-loader
    $ ./bin/webpack
    
    ERROR in ./app/javascript/packs/common.js
    Module build failed (from ./node_modules/babel-loader/lib/index.js):
    Error: Cannot find module '@babel/plugin-syntax-dynamic-import'
    Require stack:

Entonces se instalaron todos los plugins mencionados:

    $ yarn add babel-loader @babel/plugin-syntax-dynamic-import @babel/plugin-transform-destructuring @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread @babel/plugin-proposal-private-methods @babel/plugin-proposal-private-property-in-object @babel/plugin-transform-runtime @babel/plugin-transform-regenerator @babel/preset-env

Y luego quedamos con errores de Typescript:

    ERROR in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/app/javascript/config/config.service.ts
    ./app/javascript/config/config.service.ts
    [tsl] ERROR in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/app/javascript/config/config.service.ts(16,3)
          TS2554: Expected 3 arguments, but got 2.
    ERROR in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/app/javascript/context/toast-context.test.tsx
    [tsl] ERROR in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/app/javascript/context/toast-context.test.tsx(13,26)
          TS2339: Property 'toasts' does not exist on type '{}'.
    
    (...)

Intenté actualizando Yarn para ver si así se encontraba los paquetes pero nada.

    $ npm i -g yarn
    
    > yarn@1.22.19 preinstall /Users/fquintero/.nvm/versions/node/v14.17.3/lib/node_modules/yarn
    > :; (node ./preinstall.js > /dev/null 2>&1 || true)
    
    /Users/fquintero/.nvm/versions/node/v14.17.3/bin/yarnpkg -> /Users/fquintero/.nvm/versions/node/v14.17.3/lib/node_modules/yarn/bin/yarn.js
    /Users/fquintero/.nvm/versions/node/v14.17.3/bin/yarn -> /Users/fquintero/.nvm/versions/node/v14.17.3/lib/node_modules/yarn/bin/yarn.js
    + yarn@1.22.19
    updated 1 package in 0.459s

Y cuando decidí probar actualizando Typescript a la versión más reciente, otro error fatal apareció:

    $ yarn add typescript@4.7.2
    
    ERROR in ./app/javascript/ui/Root.tsx
    Module build failed (from ./node_modules/ts-loader/index.js):
    Error: Debug Failure. False expression: Non-string value passed to `ts.resolveTypeReferenceDirective`, likely by a wrapping package working with an outdated `resolveTypeReferenceDirectives` signature. This is probably not a problem in TS itself.
        at Object.resolveTypeReferenceDirective (/Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/typescript/lib/typescript.js:42530:18)
        at /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/ts-loader/dist/servicesHost.js:373:77
        at /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/ts-loader/dist/servicesHost.js:95:142
        at Array.map (<anonymous>)
        at Object.resolveTypeReferenceDirectives (/Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/ts-loader/dist/servicesHost.js:95:125)

Así que vamos a probar solo actualizar Webpacker con la versión de Rails 2.7.1

