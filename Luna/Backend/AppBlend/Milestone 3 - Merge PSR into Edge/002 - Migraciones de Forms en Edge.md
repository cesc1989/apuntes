# Migraciones de Forms en Edge

En esta etapa hay que copiar las migraciones de Patient Forms DB a Edge. Deben cumplir las reglas de [[Notas sobre Logical Replication]] así que tocó ser algo cuidadoso.

## Migraciones clave para la tabla forms

En el proyecto patient forms backend, el estado actual de la tabla forms se logró después de varias migraciones. Aquí las refiero:

- `db/migrate/20180801203056_create_forms.rb`
- `db/migrate/20180814213036_add_uuid_to_forms.rb`
	- Aquí se agrega el campo `uuid` y se le pone un índice
- `db/migrate/20181211135732_add_self_reference_to_forms.rb`
	- En esta es donde se hace la auto referencia con la columna `onboarding_id`
- `db/migrate/20190114163551_add_indexes_to_uuids.rb`
	- Aquí se agregan indices a los campos `status_uuid` y `results_uuid`
-  `db/migrate/20190114163906_add_summary_uuid_to_forms.rb`
	- Se agrega el campo `summary_uuid` con índice.

> [!Info]
> Y esta es la primera migración que escribí yo para Luna
> 
> `db/migrate/20190520192548_add_persist_draft_until_to_form.rb`

## Extra

Migraciones variadas en Rails.

Para agregar llave foránea:
```ruby
add_foreign_key :forms, :form_types
```

Para agregar llave foránea con columna diferente:
```ruby
add_foreign_key :forms, :psr_patients, column: :patient_id
```

Llave foránea con restricciones:
```ruby
add_foreign_key :forms, :form_types, column: :form_type_id, on_delete: :nullify
```

Para quitar llave foránea:
```ruby
remove_foreign_key :forms, :form_types
```

Enlaces:

- [add_foreign_key](https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_foreign_key)
- [remove_foreign_key](https://apidock.com/rails/v7.1.3.2/ActiveRecord/ConnectionAdapters/SchemaStatements/remove_foreign_key)

Etiquetas: #rails_migrations 