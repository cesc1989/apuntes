# Apuntes Ciclo 01

## Migraciones reversibles

Cuando se usa el método `change` en una migración hay que usar las macros que so reversibles.

Quería hacer que el valor por defecto del campo `kind` del modelo Appointment fuera cero. Lo iba a hacer en una migración usando `change_column` pero esa es una forma no reversible si lo usará dentro de `change` así:
```ruby
class AddDefaultToAppointmentKind < ActiveRecord::Migration[7.1]
	def change
	  change_column :appointments, :kind, :integer, default: 0
	end
end
```

Para hacerla reversible al hacer un rollback, se hace mejor así:
```ruby
class AddDefaultToAppointmentKind < ActiveRecord::Migration[7.1]
  def up
    change_column :appointments, :kind, :integer, default: 0
  end

  def down
    change_column :appointments, :kind, :integer
  end
end
```

Visto en [Stack Overflow](https://stackoverflow.com/a/22799064/1407371).