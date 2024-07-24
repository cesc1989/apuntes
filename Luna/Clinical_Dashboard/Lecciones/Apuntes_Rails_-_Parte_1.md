# Apuntes Rails - Parte 1

## Error de ActiveRecord::ConcurrentMigrationError

La aplicación se despliega en varios contenedores Docker, me imagino que el comando `rails db:migrate` se corre varias veces. Según [este artículo](https://nebulab.it/blog/the-strange-case-of-activerecord-concurrentmigrationerror/), esa sería la causa.

La solución propuesta es crear una rake task que ignore dicho error:

    # lib/tasks/migrate_ignore_concurrent.rake
    namespace :db do
      namespace :migrate do
        desc 'Run db:migrate but ignore ActiveRecord::ConcurrentMigrationError errors'
        task ignore_concurrent: :environment do
          begin
            Rake::Task['db:migrate'].invoke
          rescue ActiveRecord::ConcurrentMigrationError
            # Do nothing
          end
        end
      end
    end

El comando que la ejecutaría sería:

    bundle exec rake db:migrate:ignore_concurrent

Eso sí, como todo, tendría sus por menores. Si la 1era migración que está ejecutándose demora mucho y/o falla, pues las demás están destinadas a fallar.

Acá en [Heroku sugieren](https://help.heroku.com/UYH8N2WW/why-do-i-receive-activerecord-concurrentmigrationerror-when-running-rails-migrations-using-pgbouncer) desactivar `advisory_locks` para producción.

En [el PR que introduce](https://github.com/rails/rails/pull/22122) el cambio de `advisory_locks` el autor dice:

> Attempting to run a migration while another one is in process will
> raise a ConcurrentMigrationError instead of attempting to run in
> parallel with undefined behavior. **This could be rescued and**
> **the migration could exit cleanly instead**. Perhaps as a configuration
> option?

En [este otro artículo](https://blog.saeloun.com/2019/09/09/rails-6-disable-advisory-locks.html) explican con más detalle y al final sugieren la forma como dicen en Heroku, modificando el archivo `database.yml`.


## Servidor para peticiones falsas de integraciones

Para POCs, entre otras cosas, se han integrado APIs de Luxe. El detalle que esto tiene es cuando se quiere probar en desarrollo. Para el caso de POC, hay limitantes con respecto a los POCs por firmar que puede haber. Se necesita que alguien de Luna genera algunos falsos para tener y probar pero se acaban, entonces otra vez toca pedirle que genere.

Muy engorroso.

Buscando, encontré [DuckRails](https://github.com/iridakos/duckrails/wiki/Setup-DuckRails-natively), una aplicación Rails que puede ejecutar localmente en la cual puedo definir APIs falsos que responde con bloques estáticos o dinámicos. Sin embargo, cuando intento instalar me encuentro muchos problemas como ya [detallé en un issue](https://github.com/iridakos/duckrails/issues/72) que abrí en ese repo.


    gem install capybara-webkit -v '1.14.0' --source 'https://rubygems.org/'
    
    sh: qmake: command not found
    
    brew install qt
    
    ll /usr/local/opt/qt5/bin/
    
    export PATH=$PATH:/usr/local/opt/qt5/bin/
    
    gem install capybara-webkit -v '1.14.0' --source 'https://rubygems.org/'
    
    Project ERROR: No QtWebKit installation found. QtWebKit is no longer included with Qt 5.6, so you may need to install it separately.
    
    brew install qt55
    
    Workaruond que no sirve https://github.com/houndci/hound/issues/1320#issuecomment-282246425


## Error de RSpec mandando string vacíos en vez de nil

En esta prueba

    post(
      "#{base_api_url}/plans_of_care",
      params: { data: { physician_ids: [] } }
    )

necesitaba mandar el array cómo está, vacío. Sin embargo, RSpec decide enviarlo como un array con una cadena vacía:

    (byebug) ids
    [""]

Encuentro, [según este issue](https://github.com/rspec/rspec-rails/issues/2021), que es un error más de Rails que de RSpec y acá encuentro [la forma de solucionarlo](https://stackoverflow.com/a/44704018/1407371).


    post(
      "#{base_api_url}/plans_of_care",
      params: { data: { physician_ids: [] } },
      as: :json
    )

También sirve para que RSpec envíe el tipo de dato como es. Ejemplo:

    before do
      post(
        url,
        params: { form: { type_name: 'odi', delete_existing_answers: false } }
      )
    end

Enviará `delete_existing_answers` como `"``false``"` y no como un boolean. Para que llegue como boolean, debe ser así la petición:

    before do
      post(
        url,
        params: { form: { type_name: 'odi', delete_existing_answers: false } },
        as: :json
      )
    end


## Decodificar un JWT cuyo tiempo expiró

Normalmente, un JWT vencido no se puede decodificar:

    JsonWebToken.new(token: 'eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDQ1ODcwNjMsInByb3ZpZGVyX25hbWUiOiJBYXJvbiBTYWx5YXBvbmdzZSIsInByb3ZpZGVyX2tpbmQiOiJwa
    HlzaWNpYW4iLCJwcm92aWRlcl9pZCI6IjdlNmZlNzI4LWEyZjgtNGU3NS05OTM1LWNmMjM4NDk5OTM4NSIsInBvcnRhbF9yZWNpcGllbnRfZW1haWwiOiJmcmFuY2lzY28ucXVpbnRlcm9AaWRlYXdhcmUuY
    28iLCJkYXNoYm9hcmRfdmVyc2lvbiI6InYyIn0.saCfxyzi5qo33rfQ91lrbWpqXA1kkqmKUK-KuAPv5rM').decode
    
    Traceback (most recent call last):
    
    /Users/fquintero/.rvm/gems/ruby-2.7.1/gems/jwt-2.3.0/lib/jwt/verify.rb:41:in `verify_expiration': Signature has expired (JWT::ExpiredSignature)

Según la [documentación](https://github.com/jwt/ruby-jwt#expiration-time-claim), el indicador de vencimiento puede ser saltado:

    # Decode token without raising JWT::ExpiredSignature error
    JWT.decode token, hmac_secret, true, { verify_expiration: false, algorithm: 'HS256' }

Entonces, la solución luce así:

    token = "eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDM2NDQ4ODMsInByb3ZpZGVyX25hbWUiOiJTY3JpcHBzIEhlYWx0aCIsInByb3ZpZGVyX2tpbmQiOiJwYXJ0bmVyX2NsaW5pYyIsInByb3ZpZGVyX2lkIjoiZmVlMWY5ZWUtMTMzYy00YzE0LWE3MDktMWY2YzIzM2Y1YjNjIiwicG9ydGFsX3JlY2lwaWVudF9lbWFpbCI6InBhbGFrQGdldGx1bmEuY29tIiwiZGFzaGJvYXJkX3ZlcnNpb24iOiJ2MiJ9.YYkeFMVJHtj18atQ_-DdByMDUAPzUTU_I5Igaktm2i4"
    
    JWT.decode(token, ENV['SECRET_KEY_BASE'], true, { verify_expiration: false })


## Usando Recaptcha

Estaba configurando [recaptcha](https://github.com/ambethia/recaptcha) para la página de inicio de sesión y estaba teniendo un error con la validación. Había muchas cosas y no sabía que fallaba. Al final, lo que fallaba eran dos cosas:


1. No estaba ubicando el helper en la vista en el lugar correcto
2. No estaba usando la versión más reciente la cual solucionaba un error

La configuración es bastante sencilla cuando se quiere usar recaptcha con Devise. Está muy bien [documentado en la wiki](https://github.com/heartcombo/devise/wiki/How-To:-Use-Recaptcha-with-Devise) de Devise.

Así se debe configurar el controlador:

    # frozen_string_literal: true
    
    class Users::SessionsController < Devise::SessionsController
      prepend_before_action :check_recaptcha, only: [:create]
    
      private
    
      def check_recaptcha
        return true if verify_recaptcha(
          action: 'login',
          minimum_score: 0.5,
          secret_key: ENV['RECAPTCHA_SECRET_KEY']
        )
    
        self.resource = resource_class.new(sign_in_params)
    
        respond_with_navigational(resource) do
          flash.discard(:recaptcha_error)
    
          render :new
        end
      end
    end
    


Así la vista:

    <div class="login-form">
      <%= form_for(
          resource,
          as: resource_name,
          url: session_path(resource_name)
        ) do |f|
      %>
        <h2>Provider Portal</h2>
    
        # ...
    
        <div class="form-group">
          <%= f.submit "Log in", class: "btn" %>
        </div>
    
        <%= recaptcha_v3(action: 'login', site_key: ENV['RECAPTCHA_SITE_KEY']) %>
      <% end %>
    
      <div class="links">
        <%= render "devise/shared/links" %>
      </div>
    </div>
    

Nota que el helper en el formulario **está dentro del mismo**. El primer error que tenía era que lo estaba usando fuera del `form_for`.

Por otra parte, como no encontraba el error, decidí usar la misma versión que está en el proyecto Credentialing. Ahí no hay problema porque el parámetro `g-recaptcha-response` se envía en el cuerpo de la petición.

Acá pasaba que en la versión 5.2.1 había un error donde dicho parámetro se enviaba vacío:

    Parameters: {"authenticity_token"=>"XXXX", "user"=>{"email"=>"admin1@admin.com", "password"=>"[FILTERED]", "remember_me"=>"0"}, "commit"=>"Log in", "g-recaptcha-response"=>""}

Una vez me moví a la versión más reciente, la 5.9.0, todo quedó bien funcional:

    Parameters: {"authenticity_token"=>"XXXX", "user"=>{"email"=>"admin1@admin.com", "password"=>"[FILTERED]", "remember_me"=>"0"}, "commit"=>"Log in", "g-recaptcha-response-data"=>{"login"=>"03AGdBq27y4Cm_aHkgyV00kk86yeDsOypeSTH7Qg4hy-6kyTxT-5pp614Ni8PSBljW2K7QRPiKCresuviIE25r2o_EP5GRPISCsAhMYvbINrgqlm54ltieDuOiG8XQZW54CF5yGFJxzfLobMe142kn1-MehdM0AnIZ7OmyeOOSSq13pzwJH-5a1I1zbuR2lfmpOJS4LUcgYUBNHgk_tWMHH5FCxje54-z2lPSJeNaI5FEB_U6sV9Zm0s4j0-pzI13e4o0LqtoaI2HgnZya4AHOzLaXpiu3tA0BxocpjSSFs86L8TK14kMVdTzve5i1leIk5bn_xka7q5fUw_sdwtztiF7RVKQ1bsIgFIRwdFo0FV8Fpx96gV7c32J1Y_rFHD5Buqv8Inx6A0o01tL6pVmgcTPTmH1x6FtgSy-3FcPFB-Mp2tcLgnHJL1zwybzhKa6cEu7zW2cKgnOT"}, "g-recaptcha-response"=>""}

El funcionamiento de la gema es capturar el valor de `g-recaptcha-response` en un campo oculto llamado `g-recaptcha-response-data[action_name]`. Al actualizar al gema, se arregló el asunto y funcionó como esperaba.

**Enlaces**

- [Issue](https://github.com/ambethia/recaptcha/issues/353) donde se menciona este error.
- El [pull request](https://github.com/ambethia/recaptcha/pull/354) de la actualización.


## Las Llaves de un Hash que pasa de Resque a Clase PORO

Es bien sabido que las clases de Resque no pueden recibir objetos complejos sino primitivas nada más. Uno no envía una instancia de ActiveRecord sino que envía el ID del registro para luego buscarlo y continuar.

Con eso en mente, para el “Patients CSV Download” procedí a juntar un hash de ActionControllerParameter y lo que viene del query string (¿hay alguna diferencia?).

    Resque.enqueue(
      DownloadPatientsCsvJob,
      download_params.merge(partner: params[:partner], email: params[:email]),
      token
    )

En local, como siempre, todo funcionó bien. Sin embargo, en staging la cosa molestó y mucho.

Al parecer, cuando intentaba leer en la clase `DownloadPatientsCsvJob` la llave email como símbolo, no había nada:

    params.fetch(:email)

Luego la cambié a string pero comenzó a dar problemas en las pruebas. ¿La solución para tener contentos a todos? [Volver todas las llaves símbolos](https://coderwall.com/p/kfeo8g/convert-ruby-hash-keys-to-symbols):

    params = params.symbolize_keys

El [método](https://api.rubyonrails.org/classes/ActiveSupport/HashWithIndifferentAccess.html#method-i-symbolize_keys) `[symbolize_keys](https://api.rubyonrails.org/classes/ActiveSupport/HashWithIndifferentAccess.html#method-i-symbolize_keys)` es de ActiveSupport::HashWithIndifferentAccess, no de Ruby. En Ruby hay una forma de hacerlo pero es menos mágica. Se hace con `[Hash#transform_keys](https://ruby-doc.org/core-2.7.0/Hash.html#method-i-transform_keys)`.


## Campos de consulta con unión están ahí pero deben accederse directamente

Para el caso de la columna ROM estaba probando esta query en la consola de rails:

    r = AutoChart.select("auto_charts.id, auto_charts.rom_form, appointments.scheduled_date as date")
    .joins(appointment: [:chart, :episode])
    .where(charts: { state: 3 })
    .where.not("auto_charts.rom_form @> ?", {flexion: nil}.to_json)
    .order("appointments.scheduled_date ASC")

Eso produce una salida como esta:

    [#<AutoChart:0x0000557fd7e35998 id: 2381, rom_form: {"flexion"=>5, "extension"=>5, "displayRomForm"=>true}>,
     #<AutoChart:0x0000557fd7e35470 id: 2399, rom_form: {"flexion"=>20, "extension"=>15, "displayRomForm"=>true}>,

dónde no se ve el campo `date` que corresponde a `appointments.scheduled_date`.

El campo sí está ahí solo que el método `#``[inspect](https://ruby-doc.org/core-2.1.1/Object.html#method-i-inspect)` no puede mostrarlo. Solo puede mostrar los atributos de la clase principal (en este caso AutoChart). Para ver esos otros atributos se accede a cada objeto y ahí sí salen:

    r.each { |ac| pp "ID: #{ac.id} - ROM: #{ac.rom_form} - Scheduled: #{ac.date}" }
    
    "ID: 2381 - ROM: {\"flexion\"=>5, \"extension\"=>5, \"displayRomForm\"=>true} - Scheduled: 08/04/2021 19:30"
    "ID: 2399 - ROM: {\"flexion\"=>20, \"extension\"=>15, \"displayRomForm\"=>true} - Scheduled: 08/10/2021 13:30"

Visto en [Stack Overflow](https://stackoverflow.com/a/40037478/1407371).

## ¿Cómo enviar parámetros desde un filtro de controlador?

Visto en [Stack Overflow](https://stackoverflow.com/questions/2327682/passing-arguments-to-filters-best-practices).


## ¿Cómo retornar temprano desde un controlador?

Blog de [Arkency](https://blog.arkency.com/2014/07/4-ways-to-early-return-from-a-rails-controller/).


## CSFR Token necesario en petición POST pero no en GET

Se usa `current_user` para verificar la sesión pero en las peticiones POST se vuelve inexistente. En cambio en las GET trabaja normal, ¿por qué?

Parecer tener que ver con el token CSFR. Tal y como se menciona en la [respuesta aquí](https://stackoverflow.com/questions/18423718/rails-devise-current-user-is-nil).

Al respecto de [CSFR](https://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf) Rails dice:

> By default, Rails includes an [unobtrusive scripting adapter](https://github.com/rails/rails/blob/main/actionview/app/assets/javascripts), **which adds a header called** `**X-CSRF-Token**` **with the security token on every non-GET Ajax call**. Without this header, non-GET Ajax requests won't be accepted by Rails. When using another library to make Ajax calls, it is necessary to add the security token as a default header for Ajax calls in your library.

En `BaseApiController` tenemos esto configurado:

    protect_from_forgery with: :null_session

Así que supongo eso hace que la sesión se vuelva nula y no tenga el `current_user` disponible.

Algunos enlaces:

- [Rails: How to implement protect_from_forgery in Rails API mode](https://stackoverflow.com/questions/42795288/rails-how-to-implement-protect-from-forgery-in-rails-api-mode)
- [Understanding Rails' Protect_from_forgery](https://blog.nvisium.com/understanding-protectfromforgery)



## Creando índice compuesto en migración y modelo

Quiero crear un índice compuesto en el modelo `Dashboard` con los campos `provider_id` y `provider_email`.

Según entiendo, se [crea](https://api.rubyonrails.org/v6.1.7/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_index) así:

    add_index(:accounts, [:provider_id, :provider_email], unique: true)

Y en el modelo se usaría así ([visto aquí](https://nelsonfigueroa.dev/uniqueness-constraint-between-two-columns-in-rails/)):

    validates :provider_email, uniqueness: { scope: :provider_id, message: "Not unique" }

Sobre [esto en las guías](https://guides.rubyonrails.org/v6.1/active_record_validations.html#uniqueness) de rails.

Extra: Quitar índice:

    remove_index :dashboards, [:provider_id, :provider_email]

Más en [Saeloun](https://blog.saeloun.com/2021/03/30/rails-6-1-remove-index-if-exists.html).


## Sobre tener validaciones de active record y/o restricciones a nivel de base de datos

En este cambio, agregué una restricción a nivel de base de datos y una validación en el modelo:

    class Dashboard < ApplicationRecord
      validates :provider_email, presence: true
      validates :provider_email, uniqueness: { scope: :provider_id }
    end
    
    class AddCompositeIndexToDashboard < ActiveRecord::Migration[6.1]
      def change
        add_index :dashboards, [:provider_id, :provider_email], unique: true
      end
    end

En la revisión, Antony decía que era redundancia pero no lo veo así.

Por un lado, la restricción en la tabla hace que el error se devuelva a modo de excepción.

En cambio, la validación hace que el error se devuelva como error y se controle mejor en la aplicación.

En este [artículo de Dave Copeland](https://medium.com/@davetron5000/rails-validations-vs-postgres-check-constraints-68f8b7c9eaf5), menciona que debe usarse ambos para dar una mejor experiencia:

> If we use both the ActiveRecord validation and the check constraint, we achieve what we need: a good user experience, and the assurance of data integrity.



## Error de columna nula cuando se intenta crear referencia / llave foranea

Este es el error cuando agregaba la columna dashboard_id al modelo Activity:

    == 20221214170242 AddDashboardIdToActivity: migrating =========================
    -- add_reference(:activities, :dashboard, {:null=>false, :type=>:uuid, :foreign_key=>true})
    rails aborted!
    StandardError: An error has occurred, this and all later migrations canceled:
    
    PG::NotNullViolation: ERROR:  column "dashboard_id" of relation "activities" contains null values
    /Users/fquintero/projects/luna-project/clinical-dashboard-backend/db/migrate/20221214170242_add_dashboard_id_to_activity.rb:3:in `change'
    
    Caused by:
    ActiveRecord::NotNullViolation: PG::NotNullViolation: ERROR:  column "dashboard_id" of relation "activities" contains null values
    /Users/fquintero/projects/luna-project/clinical-dashboard-backend/db/migrate/20221214170242_add_dashboard_id_to_activity.rb:3:in `change'

La migración:

    add_reference :activities, :dashboard, type: :uuid, foreign_key: true, null: false

En una [respuesta en Stack Overflow](https://stackoverflow.com/a/63298670/1407371) que ya había votado, dice que hay que quitar la restricción de nulo. No se explica el por qué pero permite que se ejecute la migración correctamente.

Nueva migración:

    add_reference :activities, :dashboard, type: :uuid, foreign_key: true

Revisé otras llaves foráneas en otros proyectos y parece que no es necesario decirle que tenga restricción de no nulo.


    # Table name: forms
    #
    #  id                          :bigint(8)        not null, primary key
    #  patient_id                  :bigint(8)
    
    # Table name: answers
    #
    #  id               :bigint(8)        not null, primary key
    #  question_id      :bigint(8)


## Fix to Assets Path Clash

To prevent `asset_path` clash between Backend and Provider Portal, we set a prefix in Provider Portal to make the `asset_path` unique.

In file `config/environments/staging.rb`:

    Rails.application.configure do
      # (...)
      
      config.assets.prefix = '/provider-portal-assets'
    end

