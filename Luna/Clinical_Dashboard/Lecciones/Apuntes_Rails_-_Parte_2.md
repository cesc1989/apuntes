# Apuntes Rails - Parte 2

# Sidekiq no recibe instancias de clases. Volver ActionController::Parameters a un Hash

Las librerías de trabajos en segundo plano [no admiten objetos que no sean primitivas](https://github.com/mperham/sidekiq/wiki/Best-Practices#1-make-your-job-parameters-small-and-simple). 

Normalmente solo admite:

- Integer
- String
- Boolean
- Array
- Hash

Enviar instancias de clases no es ideal.

Entonces cuando se quieren mandar parámetros del controlador al worker, hay que convertirlos a un Hash así:
```ruby
download_params.merge(partner: params[:partner], email: params[:email]).to_h,
```

Así se menciona en la documentación de [ActionController::Parameters#to_h](https://api.rubyonrails.org/classes/ActionController/Parameters.html#method-i-to_h).

- `#to_h` devuelve una instancia de `ActiveSupport::HashWithIndifferentAccess`
- `#to_hash` devuelve una instancia de `Hash`

## Actualización 08/08/2023

Si se usa con el método `to_h` y sidekiq tiene activado el modo
```ruby
Sidekiq.strict_args!
```

Habrá error porque ese método devuelve un Hash con acceso indeferente. Toca usar la versión que devuelve una instancia de Hash.

# Crear tabla en migración solo si no existe

En SQL existe la sentencia `if not exists` para crear tablas si estas no existen en la base de datos. En rails también tenemos eso al crear tablas en migraciones.

Me pasó en Provider Portal que en alpha había unas tablas que no había ni en Omega ni en local. La forma correcta de proceder en este caso al crear la migración era hacerlas con algo como ese `if not exists`.

Afortunadamente eso existe en Rails. De rails 6 en adelante, se puede escribir una migración así:
```ruby
create_table :repo_subscriptions, if_not_exists: true do |t|
  t.string      :user_name
  t.string      :repo_name
  t.timestamps
end
```

Y si la tabla existe, la migración seguirá adelante y no habrá error.

Si no se escribe de esta forma, daría error como por ejemplo:
```bash
== 20240319234624 AddEdgePhysicianGroupFieldsToPhysicianGroup: migrating ======
    -- change_table(:clinics)
    rails aborted!
    StandardError: An error has occurred, this and all later migrations canceled:
    
    PG::DuplicateColumn: ERROR:  column "created_at" of relation "clinics" already exists
```

Visto en [https://dev.to/prathamesh/create-table-in-rails-only-if-it-does-not-exist-already-10mb](https://dev.to/prathamesh/create-table-in-rails-only-if-it-does-not-exist-already-10mb)

# Agregar errores a la instancia se limpian si se corre .valid?

Estaba haciendo esto:
```ruby
# 1. agregar un error personalizado
workout.errors.add(:base, "exercise_id is not assigned to workout or start_time is lesser than end_time")

# 2. y en el controlador hacía
workout.valid?

# 3. esto por si solo tampoco sirve
workout.save
```

para que `workout.save` fallara y se devolviera la respuesta de error. Pero eso no pasa porque al ejecutar `workout.valid?` o `workout.save` se [limpian los errores custom](https://stackoverflow.com/a/43508488/1407371)!

Resulta que para lograr que este error custom no se pierda hay que hacer validaciones con contextos. ¿Qué es eso? En este [post de Arkency](https://blog.arkency.com/2014/04/mastering-rails-validations-contexts/) lo explican.