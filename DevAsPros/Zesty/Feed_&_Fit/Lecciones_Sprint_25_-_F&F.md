# Lecciones Sprint 25 - F&F

## Códigos charCode de los elementos de teclado en JS

En este [sitio se pueden ver interactivamente](https://www.cambiaresearch.com/articles/15/javascript-char-codes-key-codes) sin tener que hacer un `console.log`

Para el caso de este sprint quería saber el rango que comprende los `charCode` para las flechas de desplazamiento:

![](https://paper-attachments.dropbox.com/s_8D26011A316EDB908489582FF6D3265C6FE4DE421F21B46801251095F8C607EC_1555359592422_image.png)



## Autocomplete off en formularios Rails

Parece que siempre hay que envolver otras opciones, o los atributos extra del formulario con el hash `html:{}`


    = form_tag admin_sessions_path, html: { autocomplete: "off" } do
    
    = form_for @user, html: { autocomplete: "off" } do |f|

También se puede a nivel de campo:


    = f.text_field :foo, autocomplete: 'off'

Visto en [Stack Overflow](https://stackoverflow.com/a/34870829/1407371).


## Heroku para varios entornos == varios remotos en git

La aplicación solo tiene los tres entornos por defecto: *development, test y production*. Para poder tener *staging y production* estamos usando el enfoque donde las configuraciones o llaves privadas se cargan desde variables de entorno y no por el entorno extra(ejemplo, staging).

En términos de entornos, para Heroku se trata de repositorios remotos. De este modo, para el entorno de producción y staging tenemos así:


    $ git remote -v
    origin        git@bitbucket.org:devaspros/feed-fit.git (fetch)
    origin        git@bitbucket.org:devaspros/feed-fit.git (push)
    production        https://git.heroku.com/feedandfit.git (fetch)
    production        https://git.heroku.com/feedandfit.git (push)
    staging        https://git.heroku.com/feedandfitst.git (fetch)
    staging        https://git.heroku.com/feedandfitst.git (push)

Así que cuando necesito ejecutar comandos especificos en alguno de los servidores, tengo que especificar el remoto o la app:


    # Especificando remoto
    heroku config --remote staging
    heroku config --remote production
    
    # Especificando app
    
    heroku config --app feedandfitst
    heroku config --app feedandfit

