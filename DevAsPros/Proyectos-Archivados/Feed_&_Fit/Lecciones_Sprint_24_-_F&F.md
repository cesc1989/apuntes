# Lecciones Sprint 24 - F&F
Cómo [listar las variables de entorno configuradas en Heroku](https://nathanhoad.net/how-to-list-all-environment-variables-on-heroku/):


    $ heroku config


## Cómo usar el logger de Rails en módulos

Hay que [requerir la gema logger](https://stackoverflow.com/a/12724424/1407371) y se usa con notación método de clase:

    require 'logger'
    
    Rails.logger.info "Hay..!!! Im in lib"

