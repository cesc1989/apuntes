# 027 - Waitlist Availability creada con Repeat que termina el mismo día

Etiquetas: #luna_help_desk 

Caso EDG-3113

## Contexto

El reporte dice que un paciente había designado "6:30am-9:30pm for 12/4" pero quería cambiarlo a "7am-7pm". Sin embargo, la fecha no se mostraba en el calendario al acceder a "Edit Waitlist Availability".

El error se dio porque la configuración de esa fecha estaba configurada con un "on repeat" que terminaba el mismo día:

- **Summary**: 12/4: 6:30AM-9:30PM Daily until December 4, 2025
- **Start**: 2025-12-04 04:30:00 -0800
- **End**: 2025-12-04 19:30:00 -0800
- **RRule**: `INTERVAL=1;FREQ=DAILY;UNTIL=20251204T123000Z`
- **Future occurrences**: 0

Lo cual está mal desde la configuración pero que por sí solo no causaba problema. Fue una combinación de cómo se guardó el registro y cómo devuelve los `availabilities` la query de GQL.

# Problema Real

> [!Warning]
> Esto afecta tanto los creados con la combinación repeat y sin ella.

Cuando se crea un registro que termina en una hora que crea un desborde de tiempo entre la zona horaria PST y otras se llega a este caso.

Hay un método involucrado en todo esto que está en el modelo `Availability`:
```ruby
# Literally coerce the local time in one timezone to the same local time in another timezone.
# If you find yourself using this method you should truly, TRULY ask whether or not you can
# be doing something different. Please avoid using this.
def coerced_to_time_zone(tz)
end
```

Que a pesar del comentario es usado en varias partes:
- `app/graphql/mutations/add_patient_availability.rb`
- `app/graphql/mutations/update_patient_availability.rb`
- `app/graphql/types/patient.rb`

## Pruebas en Local

Me doy cuenta porque en local tengo estas dos availabilities:
```ruby
[1] monday 11:00:AM PST 08:50:PM PST {
         :id => 39033,
:day_of_week => "monday",
 :created_at => 2025-12-18 12:46:32.124264000 PST -08:00,
 :updated_at => 2025-12-18 12:46:32.124264000 PST -08:00,
 :start_time => 2025-12-15 11:00:00.000000000 PST -08:00,
   :end_time => 2025-12-15 20:50:00.000000000 PST -08:00,
  :region_id => 6,
:location_id => 171802,
 :account_id => "2088a972-aa4d-4ae2-8201-313699eed6ed",
      :rrule => "INTERVAL=1;FREQ=DAILY;COUNT=1",
     :exdate => nil,
:creation_source => "luxe"
},


[4] sunday 11:00:AM PST 09:00:PM PST {
           :id => 39032,
  :day_of_week => "sunday",
   :created_at => 2025-12-18 12:46:12.876232000 PST -08:00,
   :updated_at => 2025-12-18 12:46:12.876232000 PST -08:00,
   :start_time => 2025-12-14 11:00:00.000000000 PST -08:00,
     :end_time => 2025-12-14 21:00:00.000000000 PST -08:00,
    :region_id => 6,
  :location_id => 171802,
   :account_id => "2088a972-aa4d-4ae2-8201-313699eed6ed",
        :rrule => "INTERVAL=1;FREQ=DAILY;COUNT=1",
       :exdate => nil,
:creation_source => "luxe"
}
```

La del Lunes que termina a las 20:50 aparece en el calendario. La del domingo que termina a las 21:00 no aparece en el calendario.

Este es el caso donde la función `coerced_to_time_zone` afecta la operación que hace el tipo de Graphql:
```ruby
# app/graphql/types/patient.rb
def availabilities(coerce_to_tz: nil)
	if coerce_to_tz.present?
		object.availabilities.includes(:region).map { |a| a.coerced_to_time_zone(coerce_to_tz) }
	else
		object.availabilities
	end
end
```

# Propuestas

## Modificar `Availabilities#ranges` ❌

Claudio propuso modificar esta función y así lo hicimos pero eso no corrigió el mostrar la avail. afectada.

## Modificar los tipos de GQL ❌

Claudio sugirió modificar `app/graphql/types/patient.rb` para que no haga coerce y funcionó pero no acepté ese cambio porque podría romper funcionalidad existente.

## Validar que no se pueda crear con Repeat que termina el mismo día ❌

> [!Note]
> Ya que el caso se resolvió para el paciente y el care plan fue discharged veo conveniente prevenir que se vuelvan a crear availabilities con esta combinación.

Este es el que tiene mejor pinta hasta ahora. Al validar e impedir que se creen registros con la combinación "Repeat que termina el mismo día" se puede prevenir a futuro.

![[13.debug.cause.png]]

Estas son las variaciones que dan problemas:

- Repeat, daily, 1 day, end on 1 occurrence
	- crea este rrule: `"INTERVAL=1;FREQ=DAILY;COUNT=1"`
- Repeat, daily, 1 day, end on same date
	- crea este rrule: `FREQ=DAILY;UNTIL=20251218`

# Enlaces

Librerías frontend para el calendario.

- DevExpress React Scheduler:
	- NPM: https://www.npmjs.com/package/@devexpress/dx-react-scheduler
	- Docs: https://devexpress.github.io/devextreme-reactive/react/scheduler/docs/reference/week-view/