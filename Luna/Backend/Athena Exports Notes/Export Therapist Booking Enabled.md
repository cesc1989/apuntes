# Export de Therapist booking_enabled para Tableau

## Detalles

En la UI este campo (`booking_enabled`) se representa en dos formas:

- En el form: es un checkbox
	- Texto: "Booking Enabled (unchecked to dry out)"
- En la vista es un texto que dice: "Scheduling Active?" y muestra una ❌ o un ✅ según sea true/false.

## Configuración en S3/Glue/Athena

El worker es: `Athena::TherapistSummaryWriterWorker`.

Exporta a la carpeta: `"business-operations/therapist-summary/data.csv"`. Lo cual es llamativo.

La tabla en Athena es: `therapist_summary`.

En Omega la carpeta en el bucket no tiene subcarpetas. Solo está el archivo `data.csv`.

En Alpha la cosa cambia. No hay archivo `data.csv`. Hay un montón de carpetas de años anteriores.