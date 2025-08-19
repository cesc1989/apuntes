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

## Asignación de dueño de Organización

En el [commit](https://github.com/cesc1989/enlacito/commit/9b55971c7e486a10705e8680fb34a58c14d448de) agregué el campo `owner_id` a organización y en [este otro](https://github.com/cesc1989/enlacito/commit/9695e59677c2f46f27db2883eedd09c3d1e1b6ca) completé la asociación y agregué una rake task. Sin embargo, la asignación no está completa hasta que habilité el registro de usuarios y establezca un flujo para la creación de la organización. Es en ese momento donde se asigna al usuario en cuestión como el dueño.

Así que queda pendiente:

- Habilitar registro de usuarios
- Asignar al dueño de la organización durante el flujo del registro