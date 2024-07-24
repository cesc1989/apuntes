# Detalle con Webpacker y el ENV

Relacionado: [[Webpacker_not_reading_from_process_env_in_Producti]]

Para la salida a producción había que configurar la URL del API en variable de entorno dado a que hay 3 entornos: `[development, staging, production]`.

Sin embargo, Webpacker no puede leer del `ENV` del sistema sino de `process.env`. Este por lo general se configura en un archivo `.env`.

En [la documentación de Webpacker](https://github.com/rails/webpacker/blob/master/docs/env.md), se sugiere esta configuración cuando el servidor de desarrollo de webpack (`./bin/webpack-dev-server`) no se levante con Foreman:

    // config/webpack/environment.js
    const { environment } = require('@rails/webpacker')
    const webpack = require('webpack')
    const dotenv = require('dotenv')
    
    const dotenvFiles = [
      `.env.${process.env.NODE_ENV}.local`,
      '.env.local',
      `.env.${process.env.NODE_ENV}`,
      '.env'
    ]
    dotenvFiles.forEach((dotenvFile) => {
      dotenv.config({ path: dotenvFile, silent: true })
    })
    
    environment.plugins.prepend('Environment', new webpack.EnvironmentPlugin(JSON.parse(JSON.stringify(process.env))))
    
    module.exports = environment

Hay una contraindicación:

> using Foreman/Invoker and npm dotenv at the same time can result in confusing behavior, in that Foreman/Invoker variables take precedence over [npm dotenv](https://github.com/motdotla/dotenv) variables.
> 
> usar Foreman y npm dotenv al mismo tiempo puede resultar en comportamiento extraño, en el sentido en que las variables de Foreman toman precedencia sobre las de [npm dotenv](https://github.com/motdotla/dotenv)

Cuando ejecuto el proyecto local con `foreman start -f Procfile.dev` está leyendo la variable `API` del archivo `.env` y no de `.env.local`.

Para el proyecto de Dashboard se probó con los archivos:


- `.env.staging` y no funcionó
- `.env.production` y no funcionó
- `.env` y **SÍ FUNCIONÓ**.

Sin embargo, el archivo `.env` debe tener los valores envueltos por comilla doble y no comilla simple.

    # .env
    API="https://physicians.luna.farzoo.click/v1"

## Changing `.env` for `.env.local`

Unignoring the `.env` meant that now I have to use a `.env.local` one and wondered about the order of those env files. Here’s [the order of loading](https://github.com/bkeepers/dotenv#what-other-env-files-can-i-use) according to the Ruby dotenv gem.

