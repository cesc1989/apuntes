# Lecciones Sprint 31 - AVZ - Devise

## Devise para varios modelos

Normalmente, una aplicación en Rails tiene el modelo de usuarios de sistema de nombre `User`. Con eso en mente, casi siempre, la configuración de Devise se hace en base a ese modelo.


    $ rails generate devise [MODEL]
    $ rails generate devise User
    $ rails generate devise Admin
    # caso de avanzza
    $ rails generate devise Owner

Sin embargo, es posible [configurar Devise para más de un tipo de usuario](https://github.com/plataformatec/devise/#configuring-views). Para que para varios modelos de usuarios tengan la posibilidad de iniciar sesión, en el inicializador de Devise en `config/initializers/devise.rb` se cambia el valor de la opción `config.scoped_views` a `true`.


    # ==> Scopes configuration
    # Turn scoped views on. Before rendering "sessions/new", it will first check for
    # "users/sessions/new". It's turned off by default because it's slower if you
    # are using only default views.
    config.scoped_views = true

Entonces para generar vistas y controladores para cada tipo de modelo de usuario, se usa el comando normal que indica la documentación pero con el modelo adecuado:


    $ rails generate devise:views # => las pone en /app/views/devise
    $ rails generate devise:views users # => las pone en /app/views/users
    $ rails generate devise:views owners # => las pone en /app/views/owners

Usando la bandera `-v` se puede indicar que conjuntos de vistas generar:


    $ rails generate devise:views -v registrations # => las pone en /app/views/devise/registrations
    $ rails generate devise:views users -v registrations # => las pone en /app/views/users/registrations

Usando la bandera `-c` se puede indicar que controladores generar:


    $ rails generate devise:controllers [scope]
    $ rails generate devise:controllers users
    $ rails generate devise:controllers users -c registrations

Finalmente, se indica la ruta para que Devise sepa que hay un controlador personalizado para dicho modelo:


    devise_for :sales_clerks,
                skip: %i[sessions registrations],
                controllers: {
                  sessions: 'sales_clerk/sessions',
                  registrations: 'sales_clerk/registrations'
                }

Con la llave `skip` se indica que no use los controladores por defecto de Devise sino los que están en `sales_clerk/sessions`.

## Redirección luego de cerrar sesión cuando hay varios modelos con Devise

Erróneamente, [tenía esta configuración](https://github.com/plataformatec/devise/wiki/How-To:-Change-the-redirect-path-after-destroying-a-session-i.e.-signing-out) en los controladores base de los modelos `SalesClerk` y de `Supplier`.

Al parecer, [esta configuración](https://www.rubydoc.info/gems/devise/Devise/Controllers/Helpers#after_sign_out_path_for-instance_method) solo funciona en `ApplicationController`


    class ApplicationController < ActionController::Base
      # ... omited
    
      private
      # ... omited  
    
      def after_sign_out_path_for(resource_or_scope)
        if resource_or_scope == :owner
          root_path
        elsif resource_or_scope == :supplier
          supplier_root_path
        elsif resource_or_scope == :sales_clerk
          sales_clerk_root_path
        end
      end
    end

Según la documentación:


> Method used by sessions controller to sign out a user. You can overwrite it in your ApplicationController to provide a custom hook for a custom scope. Notice that differently from `after_sign_in_path_for` this method receives a symbol with the scope, and not the resource.
> 
> By default it is the root_path.


## Filtrar colección de registros desde parámetro en URL

El típico caso de filtrar registros pero sin tener que recurrir a una gema. Guiándome un poco de [este artículo](https://www.justinweiss.com/articles/search-and-filter-rails-models-without-bloating-your-controller/) pues opté por solucionar el filtrado de *sales_clerks* en la vista de admin usando condicionales y *scopes:*


    # Admin::SalesClerksController
    
    def index
      @sales_clerks = SalesClerk.pending
      @sales_clerks = SalesClerk.approved if params[:approved].present?
      @sales_clerks = SalesClerk.unapproved if params[:unapproved].present?
    end

Si se recurre a gemas las opciones son:

- Ransack
- [HasScope](https://github.com/plataformatec/has_scope)
- [Rack::Reducer](https://dev.to/chrisfrank/dynamically-filter-data-via-url-params-with-rackreducer-9p6)

