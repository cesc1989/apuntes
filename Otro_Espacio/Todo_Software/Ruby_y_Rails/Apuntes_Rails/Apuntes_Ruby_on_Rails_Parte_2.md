# Apuntes Ruby on Rails. Parte 2

# ¿Cómo hacer queries en campos JSONB?

En [este artículo](https://nandovieira.com/using-postgresql-and-jsonb-with-ruby-on-rails) se explica sobre el soporte de campos JSONB en PostgreSQL y cómo usarlos en Rails.

Y en [este gist](https://gist.github.com/mankind/1802dbb64fc24be33d434d593afd6221) encontré las diferentes formas de hacer consultas sobre atributos guardados en un campo de este tipo.

```ruby
# data: {"uid"=>"5", "blog"=>"recode"}

Segment.where("data @> ?", {uid: '5'}.to_json)
Segment.where("data ->> 'blog' = 'recode'")
Segment.where("data ->> 'blog' = ?", "recode")
Segment.where("data ? :key", :key => 'uid')
Segment.where("data -> :key LIKE :value", :key => 'blog, :value => "%recode%")
```


# Sobre Llaves Foráneas

## La función `add_foreign_key` para migraciones

```ruby
add_foreign_key(from_table, to_table, options = {})
```

Uso normal:
```ruby
add_foreign_key :articles, :authors
```

Especificando una columna:
```ruby
add_foreign_key :articles, :users, column: :author_id, primary_key: "lng_id"
```

## Cómo agregar una llave foránea personalizada

```ruby
class AddFooBarStoreToPeople < ActiveRecord::Migration[5.0]
	def change
		add_reference :people, :foo_bar_store, foreign_key: { to_table: :stores }
	end
end
```

Visto en [Stack Overflow](https://stackoverflow.com/a/42214988/1407371).

Así me pasó en Luna para enlazar las tablas `patients` con `patient_summary` usando `internal_id` como llave foránea apuntando a `patients`:

```ruby
create_table :patient_summaries do |t|
	t.references :internal, null: false, foreign_key: { to_table: :patients }, type: :uuid
end
```

Sobre las opciones del helper [`add_reference`](https://apidock.com/rails/v5.2.3/ActiveRecord/ConnectionAdapters/SchemaStatements/add_reference).

## Notas sobre el artículo de Thoughtbot: Have Some (Referential) Integrity with Foreign Keys

El [artículo](https://thoughtbot.com/blog/referential-integrity-with-foreign-keys).

> [!Note]
>[Referential integrity](http://en.wikipedia.org/wiki/Referential_integrity) is a relational database concept that states implied relationships among data should be enforced. Referential integrity ensures that the relationship between rows in two tables will remain synchronized during all updates and deletes.

**Add Foreign Key Constraints** 

> Rails can’t be trusted to maintain referential integrity

Migración:
```ruby
def change
	add_foreign_key :posts, :users
end
```

## Cómo agregar varias llaves foráneas en la misma tabla?

En [Stack Overflow](https://stackoverflow.com/questions/2166613/multiple-foreign-keys-referencing-the-same-table-in-ror).

Con el campo que servirá como la otra llave foránea (aparte del *nombre_tabla_id*), en el modelo escribes algo como:

```ruby
class Modelo < ApplicationRecord
	belongs_to :billing_address, class_name: 'Address'
	belongs_to :shipping_address, class_name: 'Address'
	
	# Esta forma es referenciando desde la tabla padre
	has_one :other_address, foreign_key: "other_address_id", class_name: "Address"
end
```

# Trabajando con columnas JSON / JSONB

Primero, para agregar un campo la migración luce así:
```ruby
add_column :activities, :extras, :json
```

Y hay que agregarla al controlador como cuando se usa “nested attributes”:

```ruby
params.require(:data).permit(
	:kind,
	# (...)
	extras: [
		:chart_id,
		# (...)
	]
)
```

De [ayuda](https://smiz.medium.com/rails-forms-json-columns-and-nested-parameters-778480b1a325).

# Cargar helpers de vistas en render_to_string

En credentialing tenía esto:
```ruby
WickedPdf.new.pdf_from_string(
	ActionController::Base.new.render_to_string(
		template: 'immunizations/show',
		formats: [:pdf],
		layout: 'pdf.html',
		locals: { :@immunization => @immunization }
	)
)
```

Para generar un PDF con wicked_pdf. Como esto se está cargando en una clase que no es un controlador obtenía este error:
```bash
    ActionView::Template::Error (undefined method `symptom?' for #<ActionView::Base:0x0000000000a938>):
        11:         <label>a. Chest pain</label>
        12: 
        13:         <span>
        14:           <%= symptom?(@immunization.tuberculosis_symptoms, 'Chest pain') %>
        15:         </span>
        16:       </div>
        17: 
      
    app/views/immunizations/show.pdf.erb:14
```

La solución fue incluir las clases de los helpers así:
```ruby
ac = ActionController::Base.new
ac.view_context_class.include(ActionView::Helpers, PdfExportsHelper)

WickedPdf.new.pdf_from_string(
	ac.render_to_string(
		template: 'immunizations/show',
		formats: [:pdf],
		layout: 'pdf.html',
		locals: { :@immunization => @immunization }
	)
)
```

Visto en [Stack Overflow](https://stackoverflow.com/a/71492906/1407371).

# Cómo listar o buscar un mime en la lista en Rails

Fácil. Para listarlos todos:
```ruby
  pry(main)> Mime::EXTENSION_LOOKUP.map{|x| x}
    [["html",
      #<Mime::Type:0x000000013fc66ac0 @hash=1726007835948331403, @string="text/html", @symbol=:html, @synonyms=["application/xhtml+xml"]>],
     ["xhtml",
      #<Mime::Type:0x000000013fc66ac0 @hash=1726007835948331403, @string="text/html", @symbol=:html, @synonyms=["application/xhtml+xml"]>],
     ["text", #<Mime::Type:0x000000013fc66818 @hash=4162113904334527540, @string="text/plain", @symbol=:text, @synonyms=[]>],
     ["txt", #<Mime::Type:0x000000013fc66818 @hash=4162113904334527540, @string="text/plain", @symbol=:text, @synonyms=[]>],
     ["js",
      #<Mime::Type:0x000000013fc66660
       @hash=3122902832536570120,
       @string="text/javascript",
       @symbol=:js,
       @synonyms=["application/javascript", "application/x-javascript"]>],
```

Para encontrar uno en la lista:

```ruby
pry(main)> Mime::Type.lookup_by_extension(:pdf)
=> #<Mime::Type:0x000000011cf97550 @hash=156998226094220645, @string="application/pdf", @symbol=:pdf, @synonyms=[]>
```

Visto en este [sitio web](https://westonganger.com/posts/get-a-list-of-all-currently-registered-mime-types-with-rails).

# Entendiendo las opciones render_to_string

etiquetas: #pendiente

En este apartado intento entender lo que pasa aquí:
```ruby
WickedPdf.new.pdf_from_string(
	ActionController::Base.new.render_to_string(
		template: 'medicare_requirements/show',
		formats: [:pdf],
		layout: 'pdf.html',
		locals: { :@medicare_requirement => @medicare_requirement }
	)
)
```

# Hacer algo luego de crear un registro

Hay mucho texto sobre usar con cuidado los callbacks. Mucha gente los desaconseja. Por eso en Patient Forms para la creación de `InitialAggravatingActivities` e `InitialAnswers` las hice en una clase aparte (aunque también un poco por la lógica que necesitaban) y no en un callback.

En todo caso, la forma de [hacer algo después de crear un registro](https://api.rubyonrails.org/v7.0.4.2/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit) sería así:
```ruby
after_commit :metodo_firme, on: :create
```

Sin embargo descubrí que hay formas más cortas. No sé si son solo desde Rails 6 pero bueno, ya sé que existen:
```ruby
after_create_commit :metodo_firme
```

# Cómo invocar todas las asociaciones de un modelo?

Se hace con el método de clase `reflect_on_all_associations`. Ver [docs](https://api.rubyonrails.org/v7.0.4.2/classes/ActiveRecord/Reflection/ClassMethods.html#method-i-reflect_on_all_associations).

Así lo usé en Credentialing
```ruby
@associations_names = Therapist.reflect_on_all_associations(:has_one).map(&:name)
```

# Eliminar elementos de un Hash

Hay dos formas. La forma Ruby y la forma Rails.

## Forma Ruby

En esta forma se usa el método `delete` de la clase Hash.

```ruby
{a: 1, b: 2, c: 3, d: 4}.delete(a)
=> {a: 1}
```

Pero devuelve un hash con lo que sería la combinación de la llave borrada y su valor.

[Docs](https://ruby-doc.org/core-2.7.0/Hash.html#method-i-delete).

> Deletes the key-value pair and returns the value from hsh whose key is equal to key. If the key is not found, it returns nil.

## Form Rails

En esta forma se usa el método `except` de la extensión de clase Hash. Esto está dentro de ActiveSupport. [Ver en GitHub](https://github.com/rails/rails/blob/7c70791470fc517deb7c640bead9f1b47efb5539/activesupport/lib/active_support/core_ext/hash/except.rb). 

Sería así:
```ruby
{a: 1, b: 2, c: 3, d: 4}.except(a)
=> {b: 2, c: 3, d: 4}
```

Aquí devuelve el hash original sacando la llave estipulada como parámetro.

[Docs](https://api.rubyonrails.org/classes/Hash.html#method-i-except).

> Returns a hash that includes everything except given keys.
> 
> This is useful for limiting a set of parameters to everything but a few known toggles:
> `@person.update(params[:person].except(:admin))`

# `with_options` para agrupar macros

Resulta que en Luxe el modelo Therapist tiene esto:
```ruby
with_options if: :draft? do
  validates :ssn_last_4, presence: true, on: :update
  validates :ssn_last_4, length: { is: 4 }, on: :update
end

with_options unless: :draft? do
  validates :ssn_last_4, length: { is: 4 }, presence: true
end
```

Hace un grupo de validaciones cuando el estado es `draft`. Cuando entra en este bloque las validaciones pasan en el contexto de un `update`.

Lo otro es que cuando se no se está en `draft` el therapist, se valida al crear y actualizar.

Es decir:

- Therapist en `draft`: solo validar al actualizar
- Therapist en no `draft`: valida en cualquier contexto

Quería agregar una condición donde solo el therapist `independent_contractor` se validará para ambos casos: draft y no draft.

Para lograrlo hice esto:
```ruby
with_options if: :independent_contractor? do
	with_options if: :draft?, on: :update do
		validates :ssn_last_4, presence: true
		validates :ssn_last_4, length: { is: 4 }
	end

	with_options unless: :draft? do
		validates :ssn_last_4, presence: true
		validates :ssn_last_4, length: { is: 4 }
	end
end
```

## `with_options` anidado

Según ChatGPT, cuando se anidan bloques `with_options` Rails acumula las condiciones. En este caso Rails hará las validaciones de `ssn_last_4` cuando el therapist sea `independent_contractor` y esté (o no) en `draft`.

Cuando está en otro modo (`employee`) nunca entra en este bloque de validaciones.

## `with_options` con condición con macro interna con condición

Lo anterior funcionan según las pruebas automáticas. Lo que no funciona es tener algo como esto:
```ruby
with_options if: :draft? do
	validates :national_provider_identifier, length: { is: 10 }, presence: true, on: :update

	validates :ssn_last_4, presence: true, if: proc { |t| t.independent_contractor? }
end
```

Esto aquí no funciona porque el modificador `if` del macro `validates` toma precedencia porque Rails lo acepta como lo que tiene más prioridad.

## `with_options` con lista de condicionales

Finalmente, si necesitamos varias condiciones para poder ingresar al grupo se puede pasar una lista:
```ruby
with_options if: [:draft?, :independent_contractor?] do
end
```