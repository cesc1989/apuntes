# Lecciones Rails - Therapists Forms

## Uso de `rescue_from` para capturar excepciones en las peticiones

Aplica mucho para los casos donde se usa el método `find` y el respectivo ID no existe en la tabla. Cuando eso ocurre, Rails levanta una excepción del tipo `ActiveRecord::RecordNotFound`.

Sino se captura, pues la aplicación arrojará un error 500 en vez de indicar la causa del error. Para solucionarlo se puede capturar la excepción en cada acción del(los) controlador(es) o usar el [método de clase](https://apidock.com/rails/ActiveSupport/Rescuable/ClassMethods/rescue_from) `[rescue_from](https://apidock.com/rails/ActiveSupport/Rescuable/ClassMethods/rescue_from)`.

Hay dos formas de usarlo:

1. En un módulo
2. En un controlador base

**Usando** `**rescue_from**` en un módulo


    # app/controllers/concerns/exception_handler.rb
    module ExceptionHandler
      extend ActiveSupport::Concern
    
      included do
        rescue_from ActiveRecord::RecordNotFound, with: :not_found
      end
    
      private
    
      def not_found(e)
        render json: { errors: "#{e.model} does not exist" }, status: :not_found
      end
    end
    

Al usarlo en un módulo, normalmente se usará `ActiveSupport::Concern` para permitir que luego pueda ser usado en un controlador base.


    # app/controllers/api/base_api_controller.rb
    module Api
      class BaseApiController < ApplicationController
        protect_from_forgery with: :null_session
    
        include ExceptionHandler
      end
    end
    

**Usando** `**rescue_from**` en un controlador base


    class ApplicationController < ActionController::Base
      protect_from_forgery with: :null_session
    
      rescue_from ActiveRecord::RecordNotFound, with: :not_found
    
      rescue_from ActionController::ParameterMissing do |parameter_missing|
        render json: { errors: parameter_missing.message },
               status: :unprocessable_entity
      end
    
      rescue_from RailsParam::Param::InvalidParameterError do |invalid_parameter|
        render json: { errors: invalid_parameter.message },
               status: :unprocessable_entity
      end
    
      rescue_from ActiveRecord::InvalidForeignKey, with: :cannot_delete_record
    
      private
    
      def not_found
        render json:  { errors: 'Not found' },
               status: :not_found
      end
    
      def cannot_delete_record
        render json: { errors: 'Cannot delete' },
               status: :unprocessable_entity
      end
    end

**Relacionado**

- [Understanding Ruby and Rails: Rescuable and rescue_from](https://simonecarletti.com/blog/2009/12/inside-ruby-on-rails-rescuable-and-rescue_from/)
- [Rescue exceptions DSL for plain Ruby objects with Rails](https://railsguides.net/rescue-exceptions-dsl-for-plain-ruby-objects-with-rails/)
- [rails/activesupport/lib/active_support/rescuable.rb](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/rescuable.rb#L51)
- [Stack Overflow](https://stackoverflow.com/questions/52543840/rails-5-way-to-handle-actioncontrollerparametermissing)


## Límite de Caracteres para Identificador en PostgreSQL

El [límite para un identificador](https://til.hashrocket.com/posts/8f87c65a0a-postgresqls-max-identifier-length-is-63-bytes)(nombre de tabla, columna, llaves) es de 63 bytes.

[Según la documentación de PostgreSQL](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS):


> The tokens `MY_TABLE` and `A` are examples of identifiers. They identify names of tables, columns, or other database objects, depending on the command they are used in. Therefore they are sometimes simply called “names”. Key words and identifiers have the same lexical structure, meaning that one cannot know whether a token is an identifier or a key word without knowing the language.

Acerca del límite máximo de caracteres por nombre de identificador:


> The system uses no more than `NAMEDATALEN`-1 bytes of an identifier; longer names can be written in commands, but they will be truncated. By default, `NAMEDATALEN` is 64 so the maximum identifier length is 63 bytes. If this limit is problematic, it can be raised by changing the `NAMEDATALEN` constant in `src/include/pg_config_manual.h`.


## Migración que no corrió en RAILS_ENV=test

Me ocurrió un caso como [el que describo en este](https://github.com/rails/rails/issues/28826#issuecomment-516063445) [*issue*](https://github.com/rails/rails/issues/28826#issuecomment-516063445)*:*


- Added new model and run migration
- Rolled back, changed a column name
- Run migration `rails db:migrate`
- Specs failed because the column name in the test database didn't update

La cual arreglé con:


    $ RAILS_ENV=test rails db:drop
    $ RAILS_ENV=test rails db:create
    $ RAILS_ENV=test rails db:schema:load
## Migración para remover llave foránea creada con `references`

Si en una tabla tenemos una relación mediante llave foránea que agrega lo siguiente:


- Campo `relacion_id`: ejemplo `therapist_id`
- Llave foránea
- Índice sobre la llave foránea

La mejor forma de eliminarlo es [usando la migración](https://api.rubyonrails.org/v4.2/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-remove_reference) `[remove_reference](https://api.rubyonrails.org/v4.2/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-remove_reference)`. Aunque existen también `remove_foreign_key` y `remove_index`. `remove_reference` se encarga de todo.

Para el caso de quitar la relación de *Therapist* en *Question* y *Answer* se hizo así:


    remove_reference :questions, :therapist, index: true
    remove_reference :answers, :therapist, index: true

`index: true` para quitar los índices sobre las llaves foráneas.

## Uso del Helper `radio_button_tag` sin un formulario

Para mostrar un botón radio pero sin ningún modelo ni formulario [se puede usar el](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-radio_button_tag) [*helper*](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-radio_button_tag) [](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-radio_button_tag)`[radio_button_tag](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-radio_button_tag)` el cual simplifica la generación de un elemento HTML para este fin.

Pertenece a la [clase](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html) `[ActionView::Helpers::FormTagHelper](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html)`.


## Uso de `render_to_string` por fuera de una vista o controlador

El método `render_to_string` se usa normalmente en clases que hereden de `ActionController::Base`. Cuando se quiere usar por fuera se necesita configurar la herencia o [enviar el mensaje a una instancia de](https://stackoverflow.com/questions/2678045/render-to-string-in-lib-class-not-working) `[ActionController::Base](https://stackoverflow.com/questions/2678045/render-to-string-in-lib-class-not-working)`.


> Notar que: `[ActionController](https://api.rubyonrails.org/classes/ActionController.html)` [es un módulo](https://api.rubyonrails.org/classes/ActionController.html) y `ActionController::Base` hace referencia [a la clase](https://api.rubyonrails.org/classes/ActionController/Base.html) `[Base](https://api.rubyonrails.org/classes/ActionController/Base.html)` que pertenece al namespace `ActionController`.

De no hacerlo, el error que aparecería sería similar a este:

    NoMethodError - undefined method `render_to_string' for #<MedicarePdfAttachmentUploader:0x00007faa790a32e8>

Ejemplo de cómo lograrlo en `MedicarePdfAttachmentUploader`:

    class MedicarePdfAttachmentUploader
      def generate_pdf
        WickedPdf.new.pdf_from_string(
          ActionController::Base.new.render_to_string(
            template: 'medicare_requirements/show.pdf.erb',
            layout: 'pdf.html',
            locals: { :@medicare_requirement => @medicare_requirement }
          )
        )
      end
    end

Nota también la forma en que le puedo enviar una variable de instancia a la vista. Como en este caso se está generando un archivo PDF sin navegar a una página web, pasar la variable `@medicare_requirement` que normalmente estaría en la acción del controlador, se [pasa como variable local](https://stackoverflow.com/a/37763478/1407371) mediante el *hash* `locals: {}`.

**La nueva forma de hacerlo en Rails 5**
En este [artículo de Evil Martians](https://evilmartians.com/chronicles/new-feature-in-rails-5-render-views-outside-of-actions) describen otra forma de hacerlo usando el método de clase `render` para cada controlador, simplificando más esta acción. Esta se vale de `[ActionController::Base#renderer](https://api.rubyonrails.org/classes/ActionController/Renderer.html)` para funcionar.

Teniendo en cuenta el artículo, generar dicho PDF podría ser algo como:

    MedicareRequirementsController.render(
      :show,
      assigns: { medicare_requirement: @medicare_requirement }
    )
## `ActionMailer asset_host` para imágenes en correos salientes

Para mostrar imágenes en los correos salientes desde la aplicación Rails, [hay que configurar una variable llamada](https://guides.rubyonrails.org/v5.2/action_mailer_basics.html#adding-images-in-action-mailer-views) `[asset_host](https://guides.rubyonrails.org/v5.2/action_mailer_basics.html#adding-images-in-action-mailer-views)`.

Esta se puede configurar en `config/application.rb` o por cada entorno:


    # config/environments/production.rb
    
    config.action_mailer.asset_host = 'https://success-api-staging.luna.farzoo.click'

Sin embargo, solo con esto no será suficiente cuando se está en un entorno diferente a desarrollo.

Esto es porque la configuración `[config.public_file_server.enabled](https://guides.rubyonrails.org/v5.2/configuring.html#rails-general-configuration)` está probablemente desactivada:


> `config.public_file_server.enabled` configures Rails to serve static files from the public directory. This option defaults to `true`, but in the production environment it is set to `false` because the server software (e.g. NGINX or Apache) used to run the application should serve static files instead. If you are running or testing your app in production mode using WEBrick (it is not recommended to use WEBrick in production) set the option to `true.` Otherwise, you won't be able to use page caching and request for files that exist under the public directory.

Cómo en la aplicación la configuración está de esta forma:


    # Disable serving static files from the `/public` folder by default since
    # Apache or NGINX already handles this.
    config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?

Y la variable de entorno `ENV['RAILS_SERVE_STATIC_FILES']` no estaba puesta en el contenedor, la configuración estaba desactivada y Rails no servía los archivos presentes en `public/assets`.

Una vez la activé comenzaron a aparecer las imágenes en los correos(excepto en Gmail). Ya ahí toca mirar otra solución para los estilos.

## Otra forma de mostrar imágenes en los correos salientes

Normalmente se usaría el helper de vistas `image_tag` y todo debería funcionar normalmente. En todo caso, para situaciones donde esto pueda ser no suficiente(Gmail, por ejemplo), se puede usar la estrategia de cargar la imagen como adjunto del correo.

`ActionMailer` permite cargar adjuntos y adjuntos en línea. Usando esta segunda forma, se puede mostrar imágenes dentro del cuerpo del correo:

    class CredentialingLinkMailer < ApplicationMailer
      def link
        @therapist = params[:therapist]
    
        attachments.inline['therapist-icon.svg'] = File.read(Rails.root.join('app', 'assets', 'images', 'therapist-icon.svg'))
        attachments.inline['requirement-icon.svg'] = File.read(Rails.root.join('app', 'assets', 'images', 'requirement-icon.svg'))
    
        mail(to: @therapist.email, subject: 'Luna Credentialing Starts Now...')
      end
    end
    

Y luego se pueden usar en la vista del correo:

    <td align="left" style="padding:0;Margin:0;">
      <%= image_tag(attachments['requirement-icon.svg'].url, style: 'vertical-align: middle;') %>
      <p>Bank Information</p>
    </td>


## La Clase Date en Rails agrega otra Constante

En Ruby, la clase `[Date](https://ruby-doc.org/stdlib-2.5.3/libdoc/date/rdoc/Date.html)` tiene las siguientes constantes definidas:

    require 'date'
    
    Date.constants.sort
     => [:ABBR_DAYNAMES, :ABBR_MONTHNAMES, :DAYNAMES, :ENGLAND, :GREGORIAN, :ITALY, :Infinity, :JULIAN, :MONTHNAMES]

Por su parte, [Rails modifica esta clase](https://apidock.com/rails/Date) y le agrega una nueva constante `DATE_FORMATS`:

    Date.constants.sort
    
    [:ABBR_DAYNAMES, :ABBR_MONTHNAMES, :DATE_FORMATS, :DAYNAMES, :DAYS_INTO_WEEK, :ENGLAND, :GREGORIAN, :ITALY, :Infinity, :JULIAN, :MONTHNAMES, :WEEKEND_DAYS]

Esta define unos formatos base para mostrar fechas.

Normalmente, las fechas al ser impresas con el método `#to_s` llevan la parte del año, mes y día separados por guiones medios:

    [2] pry(main)> Therapist.first.created_at.to_s
    => "2019-09-26 16:42:32 UTC"
    [3] pry(main)> Therapist.first.care_start_date.to_s
    => "2019-12-10"

Se puede aprovechar `DATE_FORMATS` para definir variaciones al usar `#to_s`:

    # config/initializers/time_formats.rb
    
    Time::DATE_FORMATS[:luna] = '%m/%d/%Y'
    Date::DATE_FORMATS[:luna] = '%m/%d/%Y'

Y ahora usando estas variaciones:

    [4] pry(main)> Therapist.first.created_at.to_s(:luna)
    => "09/26/2019"
    [5] pry(main)> Therapist.first.care_start_date.to_s(:luna)
    => "12/10/2019"


## Sobre los métodos de Mini Magick en el Signature Generator
    def generate
        image = MiniMagick::Image.new(@tempcanvas.path)
    
        image.combine_options do |cmd|
          cmd.font(FONT_PATH)
          cmd.gravity(FONT_GRAVITY)
          cmd.pointsize(FONT_SIZE)
          cmd.draw("text 0,0 '#{@signature_text.tr("'", ' ')}'")
        end
    
        image.write("#{IMAGE_STORAGE_PATH}/#{@therapist.id}-signature.png")
    
        save_to_therapist
      end

No tenía idea de dónde saqué esos métodos. Aquí ya documentamos un poquito.


- Método `draw#text` → https://rmagick.github.io/draw.html#text
- En la doc oficial de ImageMagick → https://imagemagick.org/script/command-line-options.php#draw


## Sobre mensaje de depreciación de renderizado con extensión de vista

Salía este error luego de actualizar a Rails 6.1.4.4

    DEPRECATION WARNING: Rendering actions with '.' in the name is deprecated: medicare_requirements/show.pdf.erb (called from generate_pdf at /Users/fquintero/projects/luna-project/therapist-credentialing-backend/app/uploaders/medicare_pdf_attachment_uploader.rb:35)

Para corregir, hay que quitar la extensión del partial o template e incluir la opción `formats`:
**Antes**

    WickedPdf.new.pdf_from_string(
          ActionController::Base.new.render_to_string(
            template: 'packets/show.pdf.erb',
            layout: 'pdf.html',
            locals: { :@therapist => @therapist }
          )
        )

**Después**

    WickedPdf.new.pdf_from_string(
          ActionController::Base.new.render_to_string(
            template: 'packets/show',
            formats: [:pdf],
            layout: 'pdf.html',
            locals: { :@therapist => @therapist }
          )
        )

Enlaces al respecto:

- [Stack Overflow](https://stackoverflow.com/questions/68836670/rails-6-1-4-deprecation-warning-rendering-actions-with)
- [PR donde se implementa esta depreciación](https://github.com/rails/rails/pull/39164)
- En [las guías de Rails](https://guides.rubyonrails.org/layouts_and_rendering.html#using-render) sobre el método render y sus opciones
- Un [ejemplo en un proyecto X](https://github.com/glebm/rails_email_preview/pull/86/files) de cómo hicieron el cambio


## rails assets:precompile ejecuta todos los initializers

Para el cambio de Hubspot de API Key a Access Token se cambió la variable de entorno en el archivo `config/initializers/hubspot.rb` y quedó así:

    Hubspot.configure(access_token: ENV.fetch('HUBSPOT_ACCESS_TOKEN'))

pero eso generaba una complicación en el build en circle ci:

    Step 11/13 : RUN rails assets:precompile
     ---> Running in eddf621c32f3
    rails aborted!
    KeyError: key not found: "HUBSPOT_ACCESS_TOKEN"
    /usr/src/app/config/initializers/hubspot.rb:1:in `fetch'
    /usr/src/app/config/initializers/hubspot.rb:1:in `<main>'
    /usr/src/app/config/environment.rb:5:in `<main>'

Este error pasa porque para la construcción de la imagen Docker se necesita que dicha variable esté presente. La variable estaba configurada para los entornos staging, uat y producción pero eso es independiente del build.

Entonces para solucionar tocó agregar esa variable con un valor cualquiera en el paso anterior al de assets:precompile:

    ENV HUBSPOT_ACCESS_TOKEN="access_token"
    
    RUN bundle exec rails assets:precompile

¿Por qué pasa? Pues porque Rails procura ejecutar todo el código en la carpeta de initializers y como al usar `Hash#fetch` se levanta una excepción si falta la llave, el build se veía interrumpido.

Enlaces al respecto:

- [En el subreddit de rails](https://www.reddit.com/r/rails/comments/tjyjlw/how_to_skip_initializers_while_running_rails/)

