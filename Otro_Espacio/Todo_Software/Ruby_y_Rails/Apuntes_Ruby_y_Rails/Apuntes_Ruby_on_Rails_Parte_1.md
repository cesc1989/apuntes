# Apuntes Ruby on Rails. Parte 1

## Gema ActiveRecordImport

[Gema](https://github.com/zdennis/activerecord-import).

> Activerecord-Import is a library for bulk inserting data using ActiveRecord.

Normalmente, en ActiveRecord, al crear muchos datos lo que ocurre es que se generan múltiples consultas `INSERT INTO`.

Con esta gema, todos esos `INSERT INTO` se reducen a uno.

## Gema Goldiloader

[Gema](https://github.com/salsify/goldiloader). Artículo en [Salsify](http://blog.salsify.com/engineering/automatic-eager-loading-rails)

> Wouldn't it be awesome if ActiveRecord didn't make you think about eager loading and it just did the "right" thing by default? With Goldiloader it can!

 

## Sobre los métodos de colección de id del `has_many`

Sobre el método `ActiveRecord::Associations::ClassMethods#collection_singular_ids`

Resulta que las asociaciones `has_many` entregan unos métodos para trabajar con las mismas. Uno de esos métodos es `[collection_singular_ids](https://api.rubyonrails.org/v5.2.1/classes/ActiveRecord/Associations/ClassMethods.html)`

Lo que representa que en una asociación `Blog -> Comment`. Cada instancia de Blog tendría un método `comment_ids` y `commend_ids=`

Esos métodos reciben una colección de IDs que corresponden a llaves primarias en la relación hija(la que tiene la llave foranea) y se guardan automaticamente e inmediatamente:

> The collection_singular_ids= method makes the collection contain only the objects identified by the supplied primary key values, by adding and deleting as appropriate. The changes are persisted to the database.
> Rails [collection_singular_ids](https://guides.rubyonrails.org/association_basics.html#methods-added-by-has-many-collection-singular-ids-ids)

**¿Y qué pasa?**
Me pasa que no veo el caso donde esos métodos den error o levanten excepciones.

Una alternativa es tener un método que reciba los IDs, a modo de temporal y luego si asignarlos al método de la asociación como lo describe esta respuesta en [Stack Overflow](https://stackoverflow.com/a/13426437/1407371)

## `alias` vs `alias_method`

Ver:

- [SO](https://stackoverflow.com/questions/4763121/should-i-use-alias-or-alias-method)
- [Big Binary](http://blog.bigbinary.com/2012/01/08/alias-vs-alias-method.html)
- [Ernie Miller](https://ernie.io/2014/10/23/in-defense-of-alias/)
## Helpers de vistas no están disponibles en consola

Ni en el modelo sino se incluye el módulo correspondiente.
Ver:

- [SO](https://stackoverflow.com/questions/12332899/why-do-date-helpers-not-work-in-rails-console)
- [Documentación](https://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html#method-i-distance_of_time_in_words)

Se puede usando el receptor explícito o incluyendo el módulo `include ActionView::Helpers::DateHelper`

    helper.time_ago_in_words(1.hour.ago)
    => "about 1 hour"
    
    include ActionView::Helpers::DateHelper
    time_ago_in_words(1.hour.ago)
    => "about 1 hour" 
    
## Devise password reset token in email template

See [github 3600](https://github.com/plataformatec/devise/issues/3600) and [github 2699](https://github.com/plataformatec/devise/issues/2699#issuecomment-26971265)


## Rails and jsonb type [](https://stackoverflow.com/questions/29393562/rails-and-jsonb-type-jsonb-does-not-exist)“jsonb” does not exist

Ver: [Stack Overflow](https://stackoverflow.com/questions/29393562/rails-and-jsonb-type-jsonb-does-not-exist).
Solución: actualizar postgresql. En la respuesta aceptada indican que se actualizó del 9.1 al 9.4.


## Sobre Strong Parameters

[Según las guías de Rails](http://guides.rubyonrails.org/v4.2/action_controller_overview.html#json-parameters), cuando una petición lleva la cabecera `Content-Type` con el valor `application/json` Rails convertirá los parámetros a un `params hash`.

Adicional, si `config.wrap_parameters` está configurado, se puede omitir la llave padre en el objeto JSON y esos valores serán clonados y envueltos en una llave de acuerdo al nombre del controlador:

Sin llave padre

    { "name": "acme", "address": "123 Carrot Street" }

Con llave padre luego que rails haga la conversión

    { name: "acme", address: "123 Carrot Street", company: { name: "acme", address: "123 Carrot Street" } }

[Código fuente](https://github.com/rails/rails/blob/master/actionpack/lib/action_controller/metal/params_wrapper.rb)

## Add field, foreign key constraint and index to table

Adding a field to a table isn't enough when what you want is a full reference or a foreign key, such as when doing `users:references`. In order to achieve that one can:


    add_column :table_name, :column_name, :integer
    add_foreign_key :from_table, :to_table, column: :name
    add_index :table_name, :column_name

See:

- `[add_column](https://apidock.com/rails/v4.2.7/ActiveRecord/ConnectionAdapters/SchemaStatements/add_column)`
- `[add_foreign_key](https://apidock.com/rails/v4.2.7/ActiveRecord/ConnectionAdapters/SchemaStatements/add_foreign_key)`
- `[add_index](https://apidock.com/rails/v4.2.7/ActiveRecord/ConnectionAdapters/SchemaStatements/add_index)`


## Remover tablas e índice de columna en Migración

Para quitar una tabla ya no en uso lo mejor es hacer una migración nueva y ahí agregar la sentencia `drop_table :name`.

Ver → https://stackoverflow.com/a/31657066/1407371

Para quitar un índice de una tabla en una migración, ver https://stackoverflow.com/a/62583658/1407371


## `rails db:schema:load` solo para replicación del schema por primera vez

Como es una acción **destructiva** solo debería usarse una vez o la primera vez.


> If you run `rake db:schema:load` on a production server, you'll end up deleting all your production data. This is a dangerous habit to get into.

Ver respuesta en [Stack Overflow](https://stackoverflow.com/a/5905958/1407371).

## Get an enum integer value

Get an enum integer value: [SO](http://stackoverflow.com/questions/25570209/how-get-integer-value-from-a-enum-in-rails)

    enum sale_info: { plan_1: 1, plan_2: 2, plan_3: 3, plan_4: 4, plan_5: 5 }
    
    Model.sale_infos[my_model.sale_info] # Returns the integer value

Rails 5: [SO](http://stackoverflow.com/a/26756192/1407371)

    my_model.sale_info_before_type_cast
    
    model = Model.find(123)
    model.read_attribute('sale_info') # Rails <5
    
    model.read_attribute_before_type_cast(:sale_info) # Rails 5+


## Rake tasks with arguments and default values 📰 

Sources:

- [Ruby zigzo](http://ruby.zigzo.com/2014/08/29/rake-tasks-arguments-and-default-values/)
- [Stack Overflow](https://stackoverflow.com/a/2138836/1407371)
- [Docs](https://docs.ruby-lang.org/en/2.2.0/Rake/TaskArguments.html)


    task :task_name, [:args_1, :args_2] => :environment do |task, args|
      # task is an instance of Rake::Task
      # args is an instance of Rake::TaskArguments
    end

Y se usa así:

    rake task_name[hola,mundo]


## Sobre peticiones a Rails desde JavaScript
- Sino se envía la petición desde un formulario, hay que evadir la verificación de `authenticity token`: [SO](https://stackoverflow.com/questions/16258911/rails-4-authenticity-token)
- La petición **debe llevar los headers** `**Content-Type**` **y** `**Accept**` con `application/json`. Ejemplo:
    fetch("/echo/json/",
    {
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json'
        },
        method: "POST",
        body: JSON.stringify({a: 1, b: 2})
    })

Visto en [SO](https://stackoverflow.com/questions/29775797/fetch-post-json-data)

- Sobre `ActionController::ParameterMissing (param is missing or the value is empty: )`: [SO](https://stackoverflow.com/questions/24944871/actioncontrollerparametermissing-param-is-missing-or-the-value-is-empty-film#35030988)


## Sobre *credentials* reemplazando a *secrets* en rails 5.2
- [Goodbye secrets, welcome credentials](https://medium.com/@wintermeyer/goodbye-secrets-welcome-credentials-f4709d9f4698)

El resumen de ese artículo es:

- Desde rails 5.2 en adelante, ya no se usará *secrets.yml* sino *credentials.yml.enc*
- Para editar las credenciales hay que tener la *master key* de **desencriptación**. Esta puede ser compartida pero no comiteada en el repo.
- Para editar las credenciales usamos el comando `rails credentials:edit`. Antes hay que configurar la variable $EDITOR en la consola, ejemplo: `EDITOR=nano rails credentials:edit`
- En el servidor, lo más conveniente es configurar una variable de entorno que contenga el valor de la llave maestra. `heroku config:set RAILS_MASTER_KEY=<your-master-key-here>`

**Más al respecto**

- [problema con Devise y](https://trello.com/c/YZNhOLwJ/48-error-de-assets-precompile-con-respecto-a-secretkeybase) `[assets:precompile](https://trello.com/c/YZNhOLwJ/48-error-de-assets-precompile-con-respecto-a-secretkeybase)` [al hacer deploy en Heroku](https://trello.com/c/YZNhOLwJ/48-error-de-assets-precompile-con-respecto-a-secretkeybase)
- [PR que introduce el cambio](https://github.com/rails/rails/pull/30067)
- [Engine Yard al respecto](https://www.engineyard.com/blog/rails-encrypted-credentials-on-rails-5.2)


## Configurar UUID como llave primaria 📰 

Visto en [PostgreSQL UUID as Primary Key](https://clearcove.ca/2017/08/postgres-uuid-as-primary-key-in-rails-5-1).

Run migration:

    $ rails g migration enable_pgcrypto_extension
    
    class EnablePgcryptoExtension < ActiveRecord::Migration[5.1]
      def change
        enable_extension 'pgcrypto' unless extension_enabled?('pgcrypto')
      end
    end
    

By default Rails uses `Integer` columns for foreign keys. You need to specify the `:uuid` type for foreign keys:

    # when creating a new table
    t.references :user, type: :uuid
    
    # when modifying an existing table
    add_column :books, :user_id, :uuid

Otros artículos:

- [Empowering a Rails Application With UUID as Default Primary Key](https://betterprogramming.pub/empowering-a-rails-application-with-uuid-as-default-primary-key-44cd740828e8)
- [Guías de Rails](https://edgeguides.rubyonrails.org/active_record_postgresql.html#uuid-primary-keys)
## ¿Se pueden actualizar los UUIDs de manera manual?

Respuesta corta: sí.

Respuesta larga:

- Si hay registros que referencien el UUID a cambiar, toca actualizarlos primero porque sino habría error de restricción de llave foranea.
- Si no tiene otras referencias, sí se puede actualizar normalmente.


    [17] pry(main)> Therapist.find("960f5ad7-59d6-4654-b984-99c8edae32c5").update(id: SecureRandom.uuid)
    
    Therapist Update (6.3ms)  UPDATE "therapists" SET "id" = $1, "updated_at" = $2 WHERE "therapists"."id" = $3  [["id", "e2940e0d-79d2-4d91-afeb-57eed8cf128c"], ["updated_at", "2022-07-13 14:52:58.855672"], ["id", "960f5ad7-59d6-4654-b984-99c8edae32c5"]]
      TRANSACTION (0.4ms)  COMMIT
    
    "e2940e0d-79d2-4d91-afeb-57eed8cf128c" # nuevo UUID
    "960f5ad7-59d6-4654-b984-99c8edae32c5" # antiguo UUID
## Sobre `PG::Result`

Cuando se ejecutan consultas SQL sin ActiveRecord se pueden usar los métodos:

- `ActiveRecord::Base.connection.execute`
- y
- `ActiveRecord::Base.connection.execute_query`

El primero retorna un objeto de la clase `PG::Result`(si se usa postgres) y el segundo uno de la clase `ActiveRecord::Result`

Ver:

- [Writing SQL in Rails: Speed vs. Convenience](https://blog.lunarcollective.co/writing-sql-in-rails-speed-vs-convenience-e5d8c0ec25d9)
- [PG: The Ruby PostgreSQL Driver](#)
## `ArgumentError: invalid date` al tratar de hacer `'12/24/2018'.to_date`

Un string de fecha en el formato `mm/dd/yyyy` dará error al tratar de pasarle el mensaje `.to_date`.

Ver https://apidock.com/rails/String/to_date


## API Building
- [Cómo separar documentación de API en Blueprint en varios archivos](https://trello.com/c/pckquc5f/11-variados-ruby-rails#comment-5bcfa6c72354084461857513)
- Strong Parameters siempre da `permitted?` false para los que no son escalares `(:format)`: [Ver comentario](https://trello.com/c/pckquc5f/11-variados-ruby-rails#comment-5be4a36863b06823818530d1)


## Métodos de colección de asociación has_many
- `[#collection=](https://guides.rubyonrails.org/association_basics.html#methods-added-by-has-many-collection-objects)`
- `[#collection <<](https://guides.rubyonrails.org/association_basics.html#methods-added-by-has-many-collection-object)`
- `[#collection_singular_ids](https://guides.rubyonrails.org/association_basics.html#methods-added-by-has-many-collection-singular-ids-ids)`
- `collections_attributes = []`

Para guardar elementos de una colección en el objeto padre.
`#collection=` y `#collection <<` lo hacen recibiendo objetos de active record
`#collection_singular_ids` lo hace recibiendo IDs de objetos(un array de estos). Hace la búsqueda de los registros y cuando recibe un array hace una búsqueda con WHERE IN

Probé con estos:

    Category.last.articles << Article.create([ { name: 'TV' }, { name: 'Moto' } ])
    Category.find(6).articles << Article.build([ { name: 'Mesa' }, { name: 'Celuco' } ])
    
    cat = Category.new(
     name: 'Salud',
     articles_attributes: [
      { name: 'Jerina' },
      { name: 'Termometro' }
     ]
    )
    
    puts cat.valid?
    cat.save!


## Rails Runner

En las [guías de Rails](https://guides.rubyonrails.org/command_line.html#rails-runner) dice:

> `runner` runs Ruby code in the context of Rails non-interactively.


## Diferencia entre Plugin, Gema y Engine

En [SO](https://stackoverflow.com/questions/23118472/gem-vs-plugin-vs-engine-in-ruby-on-rails)

**Plugin**: *Plugins as you knew them from Rails 2 (i.e. plugins under the* `*vendor/plugins*` *folder) were deprecated for Rails 3.2; support for it was completely removed in Rails 4.*

> from the Rails 2 universe is an **extension** of the rails application

**Gem**: *is a packaged ruby application.*

**Engine**: *Rails Engines is basically a whole Rails app that lives in the container of another one.*

## Rails Engines
> Rails `engine` is the **miniature edition** of a Rails application. It comes with an `app` folder, including controllers, models, views, routes, migrations... The difference though is, that an `engine` won't work on its own. To show what it's capable of, it needs to become part of something bigger - a part of a main Rails application.
> 
> [Source](https://dev.to/codegram/rails-engines-independent-heroes-4bdh).

Del artículo adjunto, saco que los *engines* sirven para:

- Separar subdominios de una aplicación grande y hacerlos independientes
- Debe ser independiente y solo considerar su propio dominio
- El superdominio y otros dominios no le importan
- Es una aplicación Rails completa pero independiente
- Tiene sus pruebas automatizadas pero no sabe de las del superdominio

Recursos relacionados:

- [Engines](https://guides.rubyonrails.org/engines.html) en las Guías de Rails
- [Decidim](https://github.com/decidim/decidim): un framework conformado por puro Rails engines
## Emails case insensitive en Devise

[Wiki](https://github.com/plataformatec/devise/wiki/How-To:-Use-case-insensitive-emails).

By changing `config/initializers/devise.rb`:

    ...
    config.case_insensitive_keys = [:email]
    ...

If you need to find a user (for example in a custom sessions_controller), note that `User.find_by_email(params[:email])` is case sensitive. Use `User.find_for_authentication(:email => params[:email])` instead.

## copytruncate para logrotate en rails apps

[Stack Overflow](https://stackoverflow.com/questions/4883891/ruby-on-rails-production-log-rotation/4883967#4883967).


> What is the best way to enable log rotation on a Ruby on Rails production app?

En la solución, comentan usar `copytruncate`. Esta opción de la configuración de `[logrotate](https://linux.die.net/man/8/logrotate)` lo que hace es copiar el log y vaciarlo.


> Truncate the original log file in place after creating a copy, instead of moving the old log file and optionally creating a new one.


## Add reference with custom foreign key name

[Stack Overflow](https://stackoverflow.com/questions/27809342/rails-migration-add-reference-to-table-but-different-column-name-for-foreign-ke).

En la migración:

    add_column :people, :foo_bar_store_id, :integer
    add_index :people, :foo_bar_store_id

En el modelo:

    class Person < ActiveRecord::Base
      has_one :store, foreign_key: 'foo_bar_store_id'
    end


## Module Concerning

[Bite-sized separation of concerns](https://api.rubyonrails.org/classes/Module/Concerning.html).


## Error de null byte al guardar string en PostgreSQL

Cuando se intenta guardar una cadena como:

    "Pain medication has no e\u0000ect on my pain."

arrojará un error como el siguiente:

    An ArgumentError occurred in
    
    string contains null byte

**¿Por qué pasa?**
Lo he visto pasar en Luna, Patient Forms, cuando en el frontend copian los textos de las opciones de las preguntas y estás al venir de un PDF tienen caracterés extraños.

    "Pain medication has no e�ect on my pain."
    
    "Pain prevents me from lifting heavy weights o� the �oor, but I can manage if the weights are conveniently positioned (e.g. on a table)."

¿Solución? Que frontend [corrija la cadena o quitar](https://stackoverflow.com/a/29320495/1407371) el formato ese raro:

    string.delete("\u0000")


## Método `Object#presence`

[API Docs](https://apidock.com/rails/Object/presence).

> Returns the receiver if it’s present otherwise returns nil. object.

Sirve para resolver un problema muy común:

    object.present? ? object : nil

Al usar `#presence` se reduce a:

    object.presence

Este [artículo](https://dev.to/edwardloveall/the-rails-presence-method-3h46) tiene buenos ejemplos.

## Específica la versión de Rails a usar en la CLI

Si hay varias versiones de Rails, normalmente se usará las más actualizada. De esta forma:

    rails _6.0.3.6_ new --help

Se puede indicar qué versión usar. También aplica para [Bundle](https://paper.dropbox.com/doc/Lecciones-Sprint-27-FF--BJXbamDg1utleWQRhS2lVlleAg-trVINCusFq4C1KSiVINIw#:uid=814630730986337864324287&h2=Ejecutar-bundle-indicando-la-v).

Visto en [Stack Overflow](https://stackoverflow.com/questions/379141/specifying-rails-version-to-use-when-creating-a-new-application#452458).


## Scopes que retornan false o nulo

[En la documentación](https://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html#method-i-scope).

Los Active Record scopes cuyo resultado es false o nulo, terminan retornando el scope `all`. Eso quiere decir que van a retornar todos los registros del modelo.

> The method is intended to return an `[ActiveRecord::Relation](https://api.rubyonrails.org/classes/ActiveRecord/Relation.html)` object, which is composable with other scopes. If it returns `nil` or `false`, an [all](https://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html#method-i-all) scope is returned instead.

Me pasó en Provider Portal cuando tenía esto:

    scope :recent_unsigned, -> (key) { recent.unsigned.find_by(unsigned_path: key) } 
    
    # Query PendingPlanOfCare Load (0.5ms)  SELECT "pending_plan_of_cares".* FROM "pending_plan_of_cares"

Tuve que pasarlo a un método de clase que retornara lo que necesitaba.

## Cómo dar un layout diferente a controladores de Devise?

Cuando no se configuran los controladores de Devise (mediante el comando `rails generate devise:controllers [scope]` [ver docs](https://github.com/heartcombo/devise#configuring-controllers)), los controladores de Devise usarán el mismo layout de los controladores de la aplicación.

Por eso, si en la aplicación tenemos una cabecera con opciones de cerrar sesión u otras cosas, se mostraría también en la página de inicio de sesión. Esto podría ser indeseado.

Una forma sencilla de cambiar el layout para el controlador de Devise de sesiones es:

    class ApplicationController < ActionController::Base
      layout :layout_by_resource
    
      private
    
      def layout_by_resource
        if devise_controller?
          'login'
        else
          'application'
        end
      end
    end
    

Siendo `login` un archivo en la ruta `app/views/layouts/login.html.erb`.

En la [wiki de Devise](https://github.com/heartcombo/devise/wiki/How-To:-Create-custom-layouts) mencionan más forma de hacer esta modificación.


## Cómo mostrar una fecha con el diá en forma ordinal?

[Visto en Stack Overflow](https://stackoverflow.com/a/165213/1407371).

Si quiero mostrar una fecha así:

    Generated on February 22nd, 2022

La forma normal de `strftime` no da opción de poner el día en forma ordinal (1ero, 2do, 3ero, etc).

Para lograrlo hay que usar la gema `active-support`:

    Generated on <%= date.strftime("%B #{date.day.ordinalize}, %Y") %>

