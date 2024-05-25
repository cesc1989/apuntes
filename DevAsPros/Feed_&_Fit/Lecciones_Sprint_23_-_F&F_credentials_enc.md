# Lecciones Sprint 23 - F&F: credentials.enc

## Listas de selección y sus opciones

Como [deseleccionar todas las opciones](https://stackoverflow.com/questions/2580960/deselect-all-options-in-multiple-select-with-1-option/27949208#27949208) de una lista de selección:


    select.selectedIndex = -1;

Con `selectedIndex` se puede escoger cualquiera de los `option` de una lista mediante su índice:


    $select[0].selectedIndex = 0

Con [jQuery](https://stackoverflow.com/questions/2254736/quick-way-to-clear-all-selections-on-a-multiselect-enabled-select-with-jquery/2254755#2254755):


    $select.children().prop('selected', false)

Cómo [seleccionar](https://stackoverflow.com/questions/314636/how-do-you-select-a-particular-option-in-a-select-element-in-jquery/6068322#6068322) una opción(`option`) de un `<select>` según su texto, valor, etc:


    $(`option[value=${selected}]`, $select).prop('selected', true);


## Todo sobre la configuración de CircleCI para staging y production

La clave está en usar los workflows de CircleCI 2. Con ellos se pueden determinar flujos de trabajos para los elementos definidos en la sección `jobs` y ejecutarlos solamente para una rama usando filtros.


- [Ver sobre CircleCI Workflows](https://circleci.com/docs/2.0/workflows/#scheduling-a-workflow)
- [Ver sobre CircleCI Jobs](https://circleci.com/docs/2.0/configuration-reference/#jobs)
## Rails en Producción: secrets desde ENV o desde `credentials.enc`?


- [Ayuda rápida de](https://blog.eq8.eu/til/rails-52-credentials-tricks.html) `[rails credentials:edit](https://blog.eq8.eu/til/rails-52-credentials-tricks.html)`

Lo normal que hago es tener el entorno `staging` y el entorno `production`. Cada uno con acceso a los servicios por separado.


> Si el servicio no permite hacer una distinción entre lo que es **production** y **staging**, como lo hace S3(carpetas) o Sentry(reconoce el entorno), toca crear o dos cuentas o hacer la diferenciación mediante otros mecanismos(etiquetas, prefijos).

La incomodidad nace al querer configurar Sentry y Cloudinary para una app Heroku que tendrá staging y otra producción pero que solo definen `RAILS_ENV` como **production**.

Como ahora mismo Feed & Fit solo tiene un entorno que es **production** ahora que se va a crear otra instancia de la app para **staging** está un poco enredada la situación con `credentials`. Ya que este archivo tiene las credenciales del servicio pero las aplica para un solo entorno.

Al usar `credentials` ya no se puede tener, por ejemplo:


    RAILS_ENV="staging"
    SENTRY_DSN="https://SENTRYDSN@ALGO:ID

Y esto


    RAILS_ENV="production"
    SENTRY_DSN="https://SENTRYDSN@ALGO:ID

De hecho, [varios](https://devcenter.heroku.com/articles/deploying-to-a-custom-rails-environment) [artículos](https://medium.com/@rdsubhas/ruby-in-production-lessons-learned-36d7ab726d99) abogan a favor de tener [solo un entorno](http://teotti.com/use-of-rails-environments/): **production**.


> En el artículo [“Use of Ruby on Rails Environments”](http://teotti.com/use-of-rails-environments/) hay unos truquitos chéveres para la carga de configuraciones según el entorno cuando solo es production.


    class ServerStage
      def self.current
        ENV['STAGE']
      end
    
      def self.staging?
        current == 'staging'
      end
    end
    
    # Uso
    if ServerStage.staging?
      # do stuff
    end

**¿Por qué ocurre?**
Según Heroku y otros artículos, los entornos como `staging` no deberían existir. Sino que configuraciones que dependan del entorno deben cargarse mediante variables de entorno.

**Una esperanza**
Sin embargo, el tema ha causado incomodidad porque por más que lo ideal sería solo tener un entorno producción, pues hay muchas aplicaciones que usan multientorno.


> Para no duplicar el código de config/environments/production.rb en un config/environments/staging.rb se puede poner esta línea al inicio de staging.rb


    require Rails.root.join('config', 'environments', 'production')

En este artículo del [blog de Freeletics](https://freeletics.engineering/2018/04/04/creds-multi-environment-credentials.html) se menciona [una gema](https://github.com/freeletics/creds) que da una solución a este inconveniente usando `credentials.enc`.

De momento, estoy más a favor de seguir usando `ENV[]` que usar algunas de las soluciones a multientornos usando credenciales encriptadas.

La gema luce bien e incluso hay un *snippet*  de código para implementar la solución sin la gema pero considero que a fecha de hoy(4/4/19) no vale la pena.

De hecho, [ya hay una solución](https://github.com/rails/rails/pull/33521) oficial pero [solo estará disponible a partir de Rails 6](https://github.com/rails/rails/pull/33521#issuecomment-423672106).

