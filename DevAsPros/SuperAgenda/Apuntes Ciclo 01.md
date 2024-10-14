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

## Configura zona horaria

Ya tenía esta configuración la cual funciona:
```ruby
config.time_zone = "America/Bogota"
```

Sin embargo, en las queries en la bd se veía que estaba usando UTC:
```bash
Appointment Load (0.1ms)  SELECT "appointments".* FROM "appointments" WHERE "appointments"."user_id" = ? AND "appointments"."scheduled_at" BETWEEN ? AND ?  [["user_id", 1], ["scheduled_at", "2024-10-01 05:00:00"], ["scheduled_at", "2024-11-01 04:59:59.999999"]]
```

Estaba haciendo un rango entre inicio y fin de mes pero las horas no correspondían.

Toca hacer esto en application.rb
```ruby
config.active_record.default_timezone = :local
```

Para que la BD use la timezone que se quiere:
```bash
Appointment Load (0.1ms)  SELECT "appointments".* FROM "appointments" WHERE "appointments"."user_id" = ? AND "appointments"."scheduled_at" BETWEEN ? AND ?  [["user_id", 1], ["scheduled_at", "2024-10-14 00:00:00"], ["scheduled_at", "2024-10-20 23:59:59.999999"]]
```

Visto en [Stack Overlow](https://stackoverflow.com/questions/6118779/how-to-change-default-timezone-for-active-record-in-rails).

## Usar Time en vez de DateTime cuando la hora importe

Había registros para la fecha 23 de Octubre pero no se mostraban en el calendario de día. No se mostraban por esto:
```bash
Appointment Load (0.1ms)  SELECT "appointments".* FROM "appointments" WHERE "appointments"."user_id" = ? AND "appointments"."scheduled_at" BETWEEN ? AND ?  [["user_id", 1], ["scheduled_at", "2024-10-22 19:00:00"], ["scheduled_at", "2024-10-23 18:59:59.999999"]]
```

Había un desfase de tiempo a pesar de usar `DateTime.current`.

La solución se dio al pasar a usar la clase Time:
```bash
Appointment Load (0.1ms)  SELECT "appointments".* FROM "appointments" WHERE "appointments"."user_id" = ? AND "appointments"."scheduled_at" BETWEEN ? AND ?  [["user_id", 1], ["scheduled_at", "2024-10-23 00:00:00"], ["scheduled_at", "2024-10-23 23:59:59.999999"]]
```

¿Cuál es la diferencia? A Time le importa más la precisión de la hora.

```ruby
DateTime.current
=> Mon, 14 Oct 2024 15:49:32 -0500

Time.current
=> Mon, 14 Oct 2024 15:49:35.946805000 -05 -05:00
```

Las guías de Rails me dieron la pista.

- [Time.parse](https://rails.rubystyle.guide/#time-parse)
- [all_X](https://rails.rubystyle.guide/#date-time-range)