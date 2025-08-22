# Export de Therapist `booking_enabled` y "scheduling active" para Tableau

## Detalles

Los dos valores están en la UI en Luxe.

El campo (`booking_enabled`) se ve en el formulario de Therapist. Es un checkbox con texto: "Booking Enabled (unchecked to dry out)"

En la vista el text "Scheduling Active?" muestra una ❌ o un ✅ según sea true/false. Este valor es calculado. Se calcula en base a tres campos:

- `booking_enabled`
- `status`
- `start_booking_date`

Este es el método que lo implementa en el modelo:
```ruby
def scheduling_active?
	return false unless booking_enabled?

	%w[activated pending_activation].include?(status) && (..10.days.from_now).cover?(start_booking_date)
end
```


Por el calculo de `scheduling_active`, al exportar ambos valores hará que se vean cosas como:
```
booking_enabled: true
scheduling_active: false
```

En todo caso un caso que no se verá es:
```
booking_enabled: false
scheduling_active: true
```

Porque una vez `booking_enabled` es falso, el scheduling es falso también.

## Configuración en S3/Glue/Athena

El worker es: `Athena::TherapistSummaryWriterWorker`.

Exporta a la carpeta: `"business-operations/therapist-summary/data.csv"`. Lo cual es llamativo.

La tabla en Athena es: `therapist_summary`.

En Omega la carpeta en el bucket no tiene subcarpetas. Solo está el archivo `data.csv`.

En Alpha la cosa cambia. No hay archivo `data.csv`. Hay un montón de carpetas de años anteriores.