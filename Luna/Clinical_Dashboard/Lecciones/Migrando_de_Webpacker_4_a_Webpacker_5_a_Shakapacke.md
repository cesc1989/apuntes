# Migrando de Webpacker 4 a Webpacker 5 a Shakapacker

Este proceso lo empecé porque tanto como en [+Migrando a Ruby 3](https://paper.dropbox.com/doc/Migrando-a-Ruby-3-Mx9V178363eWJuQa4z0Bw) y en [+Migración a Rails 7](https://paper.dropbox.com/doc/Migracion-a-Rails-7-qsPe5HL2bbLJuNN2Rb6Zb) tuve problemas con Webpacker porque había que subirle la versión o por cualquier otra joda. Aquí hubo problemas pero al final se pudo.

# Pasos

Para subir a Webpacker5 se usan las gemas

    gem 'webpacker', '5.0'
    # gem 'webpacker', '5.4.3'

y luego

    bundle update webpacker

también hay que

    rails webpacker:binstubs
    
    yarn upgrade @rails/webpacker@5.0
    yarn upgrade @rails/webpacker@5.4.3

pero cuando se ejecuta `./bin/webpack` pasa que:

    ERROR in Entry module not found: Error: Can't resolve 'babel-loader' in '/Users/fquintero/projects/luna-project/clinical-dashboard-backend'

el cual es el error en los otros dos procesos.

## Webpack dev server

Cuando se actualiza en cualquier versión de la 4:

    yarn upgrade webpack-dev-server@4.0
    
    yarn upgrade webpack-dev-server@4.11
    
    ./bin/webpack-dev-server

pasa que:

    The command moved into a separate package: @webpack-cli/serve
    Would you like to install serve? (That will run yarn add -D @webpack-cli/serve) (yes/NO) : n
    @webpack-cli/serve needs to be installed in order to run the command.

Si se instala:

    yarn add -D @webpack-cli/serve@1.7.0
    
    TypeError: Class constructor ServeCommand cannot be invoked without 'new'
        at runWhenInstalled (/Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/webpack-cli/bin/utils/prompt-command.js:46:9)
        at /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/webpack-cli/bin/utils/prompt-command.js:124:15
        at processTicksAndRejections (internal/process/task_queues.js:95:5)

pero se corrige:

> It turns out that when you are prompted to install @webpack-cli/serve, it doesn't install the parent dependency. So you have to do yarn add @webpack-cli and that solves the issue.

Visto en este issue https://github.com/rails/webpacker/issues/3148


    yarn add webpack-cli
    
    ./bin/webpack-dev-server

y compila.


# Shakapacker

Rails dejó webpacker por import-maps y Shackapacker es la continuación de Webpacker.


## Enlaces

- Guía: https://github.com/shakacode/shakapacker/blob/master/docs/v6_upgrade.md
- DO NOT OVERRIDE EVERYTHING. Instead merge the diff between both gems generators
- plus react: https://github.com/shakacode/shakapacker/blob/master/docs/react.md
- plus typescript: https://github.com/shakacode/shakapacker#typescript


## Pasos

    gem 'shakapacker', '6.0.0'
    yarn add shakapacker@6.0.0

y luego:

    ./bin/webpack

Pero:

    ERROR in ./app/javascript/ui/Root.tsx 21:12
    Module parse failed: Unexpected token (21:12)
    You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
    | initSentry();
    | 
    > let Fetching: React.FC = function Fetching() {
    |   return (
    |     <main className="root root--loading">
     @ ./app/javascript/packs/common.js 1:0-20

Seguí los pasos aquí https://github.com/shakacode/shakapacker#typescript para Typescript y sigue el error de arriba pero también salen errores de tipado.

Intenté la configuración de babel en el package.json

    ,
      "babel": {
        "presets": [
          "babel.config.js"
        ]
      }

pero de esa forma no encuentra el archivo y así "./babel.config.js" da el error de arriba.

Acá Fabrizio logró arreglarlo.

Después de un `yarn install` y `./bin/webpack` todo va bien pero:

    undefined method `append_javascript_pack_tag' for #<ActionView::Base:0x0000000000b5b8> excluded from capture: DSN not set
    10:44:16 web.1       |   
    10:44:16 web.1       | ActionView::Template::Error (undefined method `append_javascript_pack_tag' for #<ActionView::Base:0x0000000000b5b8>):
    10:44:16 web.1       |     2: 
    10:44:16 web.1       |     3: <div id="root"></div>
    10:44:16 web.1       |     4: 
    10:44:16 web.1       |     5: <% append_javascript_pack_tag 'common' %>
    10:44:16 web.1       |   
    10:44:16 web.1       | app/views/dashboard/index.html.erb:5

Subimos la versión de shackapacker

    gem 'shakapacker', '6.3.0'
    
    bundle update shakapacker
    
    yarn upgrade shakapacker@6.3.0

pero `./bin/webpack` falla con Typescript otra vez. Se hizo lo siguiente:


> Update any scripts that called bin/webpack or bin/webpack-dev-server to bin/webpacker or bin/webpacker-dev-server

y `./bin/webpacker` no falló.

Finalmente, no cargaba el contenido JS.

Actualizar a versión más reciente de shakapacker

    gem 'shakapacker', '6.5.0'
    
    bundle update shakapacker
    
    yarn upgrade shakapacker@6.5.0

y se corrije.

