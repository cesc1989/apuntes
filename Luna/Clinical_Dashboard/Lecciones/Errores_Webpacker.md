# Errores Webpacker

Apareció este error

    ERROR in multi (webpack)-dev-server/client?http://localhost:3035 ./app/javascript/packs/common.js
    14:56:25 webpacker.1 | Module not found: Error: Can't resolve 'babel-loader' in '/Users/fquintero/projects/luna-project/clinical-dashboard-backend'
    14:56:25 webpacker.1 | resolve 'babel-loader' in '/Users/fquintero/projects/luna-project/clinical-dashboard-backend'
    14:56:25 webpacker.1 |   Parsed request is a module
    14:56:25 webpacker.1 |   using description file: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/package.json (relative path: .)
    14:56:25 webpacker.1 |     resolve as module
    14:56:25 webpacker.1 |       /Users/fquintero/projects/luna-project/node_modules doesn't exist or is not a directory
    14:56:25 webpacker.1 |       /Users/fquintero/projects/node_modules doesn't exist or is not a directory
    14:56:25 webpacker.1 |       /Users/fquintero/node_modules doesn't exist or is not a directory
    14:56:25 webpacker.1 |       /Users/node_modules doesn't exist or is not a directory
    14:56:25 webpacker.1 |       /node_modules doesn't exist or is not a directory
    14:56:25 webpacker.1 |       looking for modules in /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules
    14:56:25 webpacker.1 |         using description file: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/package.json (relative path: ./node_modules)
    14:56:25 webpacker.1 |           using description file: /Users/fquintero/projects/luna-project/clinical-dashboard-backend/package.json (relative path: ./node_modules/babel-loader)
    14:56:25 webpacker.1 |             no extension
    14:56:25 webpacker.1 |               /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader doesn't exist
    14:56:25 webpacker.1 |             .js
    14:56:25 webpacker.1 |               /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader.js doesn't exist
    14:56:25 webpacker.1 |             .json
    14:56:25 webpacker.1 |               /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader.json doesn't exist
    14:56:25 webpacker.1 |             as directory
    14:56:25 webpacker.1 |               /Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader doesn't exist
    14:56:25 webpacker.1 | [/Users/fquintero/projects/luna-project/node_modules]
    14:56:25 webpacker.1 | [/Users/fquintero/projects/node_modules]
    14:56:25 webpacker.1 | [/Users/fquintero/node_modules]
    14:56:25 webpacker.1 | [/Users/node_modules]
    14:56:25 webpacker.1 | [/node_modules]
    14:56:25 webpacker.1 | [/Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader]
    14:56:25 webpacker.1 | [/Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader.js]
    14:56:25 webpacker.1 | [/Users/fquintero/projects/luna-project/clinical-dashboard-backend/node_modules/babel-loader.json]

El cual se solucionó con el comando `yarn upgrade`. Vimos la solución [acá (entre otras alternativas)](https://stackoverflow.com/questions/34538466/error-cannot-resolve-module-babel-loader).

