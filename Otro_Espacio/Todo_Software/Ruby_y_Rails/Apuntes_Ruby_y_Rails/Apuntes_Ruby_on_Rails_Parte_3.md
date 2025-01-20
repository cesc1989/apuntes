# Apuntes Ruby on Rails. Parte 3

# Error de Parameters

Cuando se tiene una lista de parámetros así:
```ruby
:pain_scale,
patient_attributes: [],
intake_form_attributes: [],
aggravating_activities_attributes: [],
answers_attributes: [:id, :option_choice_id],
:submitted_from
```

El último elemento suele causar este error:
```bash
onboarding_forms_controller.rb:60: syntax error, unexpected '\n', expecting =>
						:submitted_from
													 ^
/Users/francisco/projects/luna-project/patient-forms-backend/app/controllers/api/v1/onboarding_forms_controller.rb:70: syntax error, unexpected end-of-input, expecting `end' excluded from capture: DSN not set
	
SyntaxError (/Users/francisco/projects/luna-project/patient-forms-backend/app/controllers/api/v1/onboarding_forms_controller.rb:60: syntax error, unexpected '\n', expecting =>
						:submitted_from
													 ^
/Users/francisco/projects/luna-project/patient-forms-backend/app/controllers/api/v1/onboarding_forms_controller.rb:70: syntax error, unexpected end-of-input, expecting `end'):
	
onboarding_forms_controller.rb:60: syntax error, unexpected '\n', expecting =>
onboarding_forms_controller.rb:70: syntax error, unexpected end-of-input, expecting `end'
onboarding_forms_controller.rb:60: syntax error, unexpected '\n', expecting =>
onboarding_forms_controller.rb:70: syntax error, unexpected end-of-input, expecting `end'
```

¿Por qué?


# No crear migración con null false si en la tabla existen registros

Habrá un error al correr la migración porque los registros existentes tendrán esa columna con valor nulo:
```bash
Migrating to AddKindToOutbox (20231002222854)
== 20231002222854 AddKindToOutbox: migrating ==================================
-- add_column(:outbox, :kind, :string, {:null=>false})
	TRANSACTION (0.7ms)  BEGIN
	 (2.3ms)  ALTER TABLE "outbox" ADD "kind" character varying NOT NULL
	TRANSACTION (0.8ms)  ROLLBACK
rails aborted!
StandardError: An error has occurred, this and all later migrations canceled:

PG::NotNullViolation: ERROR:  column "kind" of relation "outbox" contains null values
/usr/src/app/db/migrate/20231002222854_add_kind_to_outbox.rb:3:in `change'

Caused by:
ActiveRecord::NotNullViolation: PG::NotNullViolation: ERROR:  column "kind" of relation "outbox" contains null values
/usr/src/app/db/migrate/20231002222854_add_kind_to_outbox.rb:3:in `change'

Caused by:
PG::NotNullViolation: ERROR:  column "kind" of relation "outbox" contains null values
/usr/src/app/db/migrate/20231002222854_add_kind_to_outbox.rb:3:in `change'
Tasks: TOP => db:migrate
```

En estos casos lo que hay que hacer es:

1. migración que agregue el nuevo campo
2. llenar los registros existentes con un valor para ese campo
3. agregar migración con restricción “null: false”

Con el comando `rails db:migrate:status` se puede ver hasta donde han corrido las migraciones.

Este es un ejemplo en Patient Forms:
```bash
up     20220624204226  Create settings
up     20230307135035  Add pre correction nps to form
up     20230707145353  Add nps feedback to form
up     20230921224538  Create outboxes
down    20231002222854  Add kind to outbox
down    20231004135232  Change kind in outbox to nullable
```

Las dos últimas en `down` no fueron corridas porque daba el error que pegué arriba. En local sí funcionó pero no pasaba en alpha.

Para arreglar deshice las migraciones en local (con `rails db:rollback`) y cuando en local esas dos aparecían como en alpha:
```bash
down    20231002222854  Add kind to outbox
down    20231004135232  Change kind in outbox to nullable
```

quité la restricción de la migración que agregaba el campo y eliminé la segunda porque estaba sobrando.

Enlaces:

- [Stack Overflow](https://stackoverflow.com/a/4797100/1407371)


# Asignar parámetros a objeto active record con attributes= no guarda al hacer .save

Tengo este código en Luxe:
```ruby
def update
	# if workout.update(workout_params)
	workout.attributes = workout_params
	if workout.save
		debugger
		render json: workout, status: :ok
	else
		render json: workout.errors, status: :unprocessable_entity
	end
end
```

pero en las pruebas, al intentar verificar el cambio de valores de los parámetros asignados, no funciona.

```bash
    1.1) Failure/Error: expect(workout.pain_level).to eq(1)
              
                expected: 1
                     got: 6
                     
    1.2) Failure/Error: expect(workout.completion_percent).to eq(0.77)
              
                expected: 0.77
                     got: 0.0
```

Cuando inspecciono en el debugger veo esto.

Los parámetros son permitidos:
```ruby
workout_params
#<ActionController::Parameters {"ended_at"=>"10/14/2018 11:00", "completion_percent"=>"0.77", "pain_level"=>"1"} permitted: true>
```

Los atributos del objeto a modificar:
```ruby
  workout.attributes
    {"id"=>"4a70c38b-9c75-45e9-86f6-4834a1e16365", "started_at"=>nil, "ended_at"=>nil, "completion_percent"=>0.0, "pain_level"=>8, "exercise_program_id"=>"48617728-cf5b-475c-a3a8-8a3dd2489e74", "created_at"=>Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00, "updated_at"=>Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00, "exercise_times"=>{}}
```

pero al asignar nada pasa:
```ruby
   workout.attributes = workout_params
    #<ActionController::Parameters {"ended_at"=>"10/14/2018 11:00", "completion_percent"=>"0.77", "pain_level"=>"1"} permitted: true>
    (byebug) workout.ended_at
    nil
    (byebug) workout.attributes = workout_params.to_hash
    {"ended_at"=>"10/14/2018 11:00", "completion_percent"=>"0.77", "pain_level"=>"1"}
    (byebug) workout.ended_at
    nil
```

[Esta página menciona](https://scottbartell.com/2022/04/12/set-attributes-in-active-record-rails-7/) como esta es una forma válida de asignar atributos en Rails 7 y así también [lo muestra la documentación](https://api.rubyonrails.org/v7.0/classes/ActiveModel/AttributeAssignment.html).

¿por qué falla?

# Al mezclar hashes, usa with_defaults

Tomado de [este artículo](https://andycroll.com/ruby/with_defaults-clarity-when-merging-hashes/).

En el artículo, el autor dice que en vez mezclar usando `#merge`:
```ruby
user_provided = {q: "Andy", age: 44, limit: 1}

listing_options = {order: "asc", limit: 25}.merge(user_provided)
#=> {:order=>"asc", :limit=>1, :q=>"Andy", :age=>44}
```

Se haga así, usando `#with_defaults`:
```ruby
user_provided = {q: "Andy", age: 44, limit: 1}

listing_options = user_provided.with_defaults(order: "asc", limit: 25)
#=> {:order=>"asc", :limit=>1, :q=>"Andy", :age=>44}
```

Usando `with_defaults` se es más explícito en que son valores secundarios y que lo que importa son los valores que se reciben de un usuario o petición.

En otras palabras, los por defecto son solo si no hay valores preseleccionados presentes.

Y esas intenciones quedan más claras al usar `#with_default` que al usar el otro método.

# Crear aplicación nueva con versión específica de Rails

Etiquetas: #rails_new Palabras clave: rails new.

Ejemplos con Puntapie:
```bash
rails _7.0.1_ new ../pietest -d postgresql -m ./template.rb

rails _7.0.1_ new ../pietest -d sqlite3 -m ./template.rb
```

# ActiveRecord insert para crear sin levantar callbacks ni instancias

Este método sirve para crear registros sin instanciar objetos ni tampoco disparar callbacks.

No sabía que existía hasta que lo vi en Luna:
```ruby
Patient.insert!(attrs)
```

Documentación: https://apidock.com/rails/v6.0.0/ActiveRecord/Persistence/ClassMethods/insert

Ejemplo en Cash Flow
```ruby
Expenditure.insert({description: 'prueba insert', amount: 50000, user_id: 1, category_id: 29})

	Expenditure Insert (4.5ms)  INSERT INTO "expenditures" ("description","amount","user_id","category_id","created_at","updated_at") VALUES ('prueba insert', 50000, 1, 29, STRFTIME('%Y-%m-%d %H:%M:%f', 'NOW'), STRFTIME('%Y-%m-%d %H:%M:%f', 'NOW')) ON CONFLICT  DO NOTHING

=> #<ActiveRecord::Result:0x000000010c29d210 @column_types={}, @columns=[], @hash_rows=nil, @rows=[]> 
```