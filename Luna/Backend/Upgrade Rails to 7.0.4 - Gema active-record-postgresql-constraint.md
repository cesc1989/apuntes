# Upgrade Rails to 7.0.4 - Gema active-record-postgresql-constraint

Necesito actualizar este proyecto a Rails 7.0.4 desde la versión 6.1.6. Hay algunos puntos clave antes de poder hacer la migración. Debo entender las implicaciones de algunas gemas, qué código _custom_ podría tener con Rails 7 y con Ruby 3.1 para poder hacer la actualización.

En este documento procedo a describir el caso de la gema active_record-postgresql-constraints.

## Particularidades

Edge actualmente está en:

**Rails 6.1.6**
- Objetivo: 7.0.4
- Las versiones que le siguen:
	- 6.1.6.1
	- 6.1.7
	- 6.1.7.1
	- 6.1.7.2
	- 6.1.7.3
	- ...
	- 6.1.7.8 (4 de Junio, 2024)

**Ruby 3.0.6**
- Objetivo: 3.1.0
- Las versiones que le siguen: 
	- 3.0.7
	- 3.1.0

Gemas a revisar:
- active_record-postgres-constraints -> [https://github.com/betesh/active_record-postgres-constraints?tab=readme-ov-file](https://github.com/betesh/active_record-postgres-constraints?tab=readme-ov-file)   

Enlaces
- [Página de versiones de Rails en GitHub](https://github.com/rails/rails/tags)
- [Página de versiones de Ruby](https://www.ruby-lang.org/en/downloads/releases/)

# Gema active_record-postgres-constraints

La explicación de por qué la archivaron -> [https://github.com/betesh/active_record-postgres-constraints/issues/37#issuecomment-824539916](https://github.com/betesh/active_record-postgres-constraints/issues/37#issuecomment-824539916)

Readme de la versión compatible con rails 6.1 -> [https://github.com/betesh/active_record-postgres-constraints/tree/support-rails-6.1](https://github.com/betesh/active_record-postgres-constraints/tree/support-rails-6.1)

Describe que añade los siguientes métodos:
- t.check_constraint
- add_check_constraint
- remove_check_constraint
- t.exclude_constraint
- add_exclude_constraint
- remove_exclude_constraint

Haciendo una búsqueda en el editor encuentro estos resultados:
- 9 resultados para `check_constraint` (tanto en `t.` como `add_`)
    - 4 de ellos son en el mismo archivo: schema.rb
- 26 resultados para `exclude_constraint` (tanto en `t.` como `add_`)
    - 4 de ellos son en el mismo archivo: schema.rb

Pull Request de Rails donde agregan `add_check_constraint` -> [https://github.com/rails/rails/pull/31323](https://github.com/rails/rails/pull/31323)

Pull request de Rails donde agregan `add_exclusion_constraint` -> [https://github.com/rails/rails/pull/40224](https://github.com/rails/rails/pull/40224)

Post en el foro de Rails donde se menciona el pr anterior [https://discuss.rubyonrails.org/t/postgres-exclusion-constraint-support/78120](https://discuss.rubyonrails.org/t/postgres-exclusion-constraint-support/78120)

> nota que Anthony participa del foro y el pr

## Atención

- La gema que está en Backend es un fork de la original.
- En ese fork se agregó un commit para ampliar el soporte de Rails hasta la versión 7.1
    - Commit [https://github.com/lunacare/active_record-postgres-constraints/commit/bb0f224eb2713e03cca21fb5b77ae354197e7550](https://github.com/lunacare/active_record-postgres-constraints/commit/bb0f224eb2713e03cca21fb5b77ae354197e7550)

## Preguntas

- Q: ¿El uso de `add_check_constraint` es de la gema o de Rails?
	- R: Es de Rails. Hay una migración que usa la sintaxis de la gema pero actualmente no se usa más `rails db:migrate` al clonar el proyecto. No debería molestar.
- Q: ¿Cuál es la característica que edge usa exclusivamente, exclude_constraint?
    - R: sí, así lo menciona Anthony en este [comentario](https://discuss.rubyonrails.org/t/postgres-exclusion-constraint-support/78120/2) 

## Enlaces

- Saeloun sobre check_constraints [https://blog.saeloun.com/2021/01/08/rails-6-check-constraints-database-migrations/](https://blog.saeloun.com/2021/01/08/rails-6-check-constraints-database-migrations/)
- Boring Rails [https://boringrails.com/tips/rails-check-constraints-database-validations](https://boringrails.com/tips/rails-check-constraints-database-validations)
- API Docs add_check_constraint [https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_check_constraint](https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_check_constraint)
    - Rails API [https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_check_constraint](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_check_constraint)
- API Docs remove_check_constraint [https://apidock.com/rails/v7.0.0/ActiveRecord/ConnectionAdapters/SchemaStatements/remove_check_constraint](https://apidock.com/rails/v7.0.0/ActiveRecord/ConnectionAdapters/SchemaStatements/remove_check_constraint)
- Guides exclusion_constraint [https://guides.rubyonrails.org/active_record_postgresql.html#exclusion-constraints](https://guides.rubyonrails.org/active_record_postgresql.html#exclusion-constraints)

# Línea de tiempo de la gema active_record-postgres-constraints y Rails agregando soporte nativo

## check_constraint

- **3 de Diciembre de 2017:** abren PR para agregar add_check_constraint
- **3 de Junio de 2020:** se [mezcla](https://github.com/rails/rails/pull/31323#event-3400435896) el PR.

## exclusion_constraint

- **12 de Septiembre de 2020:** PR que agrega add_exclusion_constraint
- **2 de Junio de 2021:** [este post](https://discuss.rubyonrails.org/t/postgres-exclusion-constraint-support/78120) en el foro llama la atención del PR de `add_exclusion_constraint`
- **30 de Marzo de 2022:** Anthony actualizó el fork para soportar rails 7.1
- **14 de Junio de 2022:** pr fue [mezclado](https://github.com/rails/rails/pull/40224#issuecomment-1155151178).

# Sintaxis entre la gema active_record-postgres-constraints y Rails

## Check Constraint

### Sintaxis de la gema

```ruby
t.check_constraint(name_or_conditions, conditions = nil)
add_check_constraint(table_name, name, conditions)
remove_check_constraint(table, name, conditions = nil)
```

### Rails al crear tabla

```ruby
check_constraint(*args, **options)

# Ejemplo
# check_constraint [constraint_name], [constraint]
check_constraint "price_check", "price > 100"
```

Ver en [los docs](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/Table.html#method-i-check_constraint) para el método de instancia.

### Rails en tablas existentes

```ruby
add_check_constraint(table_name, expression, if_not_exists: false, **options)

# Ejemplo
# add_check_constraint :table_name, :constraint_condition, name: "constraint_name"
add_check_constraint :books, "price > 100", name: "price_check"

remove_check_constraint(table_name, expression = nil, if_exists: false, **options)

# Ejemplo
# remove_check_constraint :table_name, name: "constraint_name"
remove_check_constraint :books, name: "price_check"
```

El autor de la gema dice que:

> Rails 6.1 (...) expression must be a String. Hash and Array are no longer supported.

**Enlaces:**
- [remove_check_constraint](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-remove_check_constraint)

###  Preguntas

- Q: ¿Es lo que dice el autor sobre _expression must be a String_ cierto aún?
    - R: Sí, la [documentación](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_check_constraint) así lo indica.

## Exclusion o Exclude Constraint

### Sintaxis de la gema

```ruby
t.exclude_constraint using: :gist, 'tsrange("from", "to")' => :overlaps, room_id: :equals

add_exclude_constraint :booking, using: :gist, 'tsrange("from", "to")' => :overlaps, room_id: :equals

# If you don't need it to be reversible:
remove_exclude_constraint :booking

# If you need it to be reversible (Recommended):
remove_exclude_constraint :booking, using: :gist, 'tsrange("from", "to")' => :overlaps, room_id: :equals
```

### Rails al crear tabla

```ruby
t.exclusion_constraint("price WITH =, availability_range WITH &&", using: :gist, name: "price_check")
```

### Rails en tablas existentes

```ruby
add_exclusion_constraint(table_name, expression, **options) 

# Ejemplo
add_exclusion_constraint :products, "price WITH =, availability_range WITH &&", using: :gist, name: "price_check"

remove_exclusion_constraint(table_name, expression = nil, **options)

# Ejemplo
remove_exclusion_constraint :products, name: "price_check"
```

Enlaces:
- [exclusion_constraint](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/PostgreSQL/Table.html#method-i-exclusion_constraint)
- [add_exclusion_constratint](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/PostgreSQL/SchemaStatements.html#method-i-add_exclusion_constraint)
- [remove_exclusion_constraint](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/PostgreSQL/SchemaStatements.html#method-i-remove_exclusion_constraint)

# ¿Qué migraciones usan la sintaxis de la gema?

## Usando add_check_constraint

Migración: `create_protocols_tables`
```ruby
t.check_constraint "(number > 0) AND (beginning_days_after_trigger_date >= 0)"
```

Esta es la única ocurrencia que usa la sintaxis de la gema al crear una tabla. Los otros resultados usan la sintaxis de Rails claramente mediante el macro de migraciones (`add_check_constraint`).

La sintaxis de la macro es, en resumen:
```ruby
add_check_constraint :table, "constraint", name: "constraint_name"
```

Sin embargo, esta constraint al parecer fue eliminada en una migración posterior.

```ruby
t.exclude_constraint :protocol_phases_802620237, using: :gist, relative_period: :overlaps, protocol_id: :equals
```

Y vemos que usa la sintaxis de la gema.

## Usando add_exclude_constraint

En el archivo schema.rb hay seis resultados que usan la sintaxis de la gema. Si se quita la gema, al ejecutar `rails db:schema:load` esto fallará.

### Preguntas

- Q: ¿Puedo hacer el upgrade de Rails sin quitar la gema?