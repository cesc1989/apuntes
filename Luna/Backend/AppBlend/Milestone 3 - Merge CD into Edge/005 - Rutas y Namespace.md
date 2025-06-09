# Rutas y Namespace

## devise_for

Tenía esta configuración para Devise con el modelo User:
```ruby
namespace :clinical_dashboard, path: "/" do
  devise_for :users
end
```

Eso daba este error:
```
uninitialized constant User (NameError)

Object.const_get(camel_cased_word) ^^^^^^^^^^

Did you mean? Uber
```

Lo que significaba que Devise no podía encontrar la clase `User` porque está dentro de un namespace.

Probé varias cosas. Ninguna funcionó:
```ruby
devise_for :users, module: :clinical_dashboard

devise_for :users, path: "clinical_dashboard"

devise_for :users, module: "clinical_dashboard", singular: :user
```

Pero había entendido mal todo y no leí una parte de la documentación.

La solución es esta:
```ruby
devise_for :users, class_name: "ClinicalDashboard::User", module: :devise
```

Esto dicen los docs de Devise:

> class_name: set up a different class to be looked up by devise, if it cannot be
  properly found by the route name.

## current_user

Dado a que el modelo user está dentro del namespace ClinicalDashboard ya no tengo disponible `current_user`:
```
undefined local variable or method `current_user'`
```

La solución es usar el nombre según el namespace + el modelo:
```ruby
current_clinical_dashboard_user
```

Este helper se define en el módulo `Devise::Controllers::Helpers`. Lo hace así:
```ruby
def current_#{mapping}
	@current_#{mapping} ||= warden.authenticate(scope: :#{mapping})
end
```

Y podemos ver los mappings en la consola de rails así:
```ruby
Devise.mappings
=> {:clinical_dashboard_user=>
  #<Devise::Mapping:0x000000012245dfd0
   @class_name="ClinicalDashboard::User",
   @controllers={:sessions=>"devise/sessions"},
   @failure_app=Devise::FailureApp,
   @format=nil,
   @klass=#<Devise::Getter:0x000000012245dc38 @name="ClinicalDashboard::User">,
   @modules=[:database_authenticatable, :validatable, :timeoutable, :trackable],
   @path="users",
   @path_names={:registration=>"", :new=>"new", :edit=>"edit", :sign_in=>"sign_in", :sign_out=>"sign_out"},
   @path_prefix="/",
   @router_name=nil,
   @routes=[:session],
   @scoped_path="clinical_dashboard/users",
   @sign_out_via=:delete,
   @singular=:clinical_dashboard_user,
   @strategies=[:database_authenticatable],
   @used_helpers=[:session],
   @used_routes=[:session]>,
```