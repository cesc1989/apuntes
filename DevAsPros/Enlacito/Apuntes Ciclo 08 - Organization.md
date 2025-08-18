# Apuntes Ciclo 08 - Organization

## Migración de agregar columna para llave foránea

Se puede lograr usando tres migraciones. Ejemplo para cuando agregué el campo `owner_id` a la tabla `organiations`.

```ruby
class AddOwnerIdToOrganization < ActiveRecord::Migration[7.1]
  def change
    add_column :organizations, :owner_id, :bigint
    add_foreign_key :organizations, :users, column: :owner_id
    add_index :organizations, :owner_id
  end
end
```

También se puede de esta forma:
```ruby
add_reference :organizations, :owner, foreign_key: { to_table: :users }, index: true
```

