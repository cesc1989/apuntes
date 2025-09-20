# 003 - Export Therapist Inactive Reason to therapist-forward-fill table

Etiquetas: #luna_help_desk 

## Pruebas

Para disparar manualmente el worker en una consola de Rails:
```ruby
Athena::TherapistForwardFillWriterWorker.perform_async
```

Queries de prueba en Athena:
```sql
SELECT *
FROM "business-operations"."therapist_forward_fill"
where inactive_reason_key is not null
limit 20;
```

```sql
SELECT *
FROM "business-operations"."therapist_forward_fill" tfi
where tfi.inactive_reason_key <> ''
order by tfi.partition_0 desc
limit 200;
```

```sql
SELECT *
FROM "business-operations"."therapist_forward_fill" tfi
order by tfi.partition_0 desc
limit 200;
```

## Backfill de datos históricos

Resulta que este export se da en particiones. Cada vez que se ejecuta se crea una carpeta con la fecha actual:
```ruby
def writer_params
	partition_name = Time.now.in_time_zone("UTC").iso8601.gsub(":", "_")
	{
		bucket_name: ENV["DATA_LAKE_S3_BUCKET"],
		object_path: "business-operations/therapist-forward-fill/#{partition_name}/data.csv",
		sql_query: query,
```

Así que al completar el export de la forma normal solo habrá datos recientes. Para poder completar los datos históricos hay que completar un backfill. Este proceso es delicado porque hay que actualizar todos los archivos CSV que existen en el bucket, comprobar que la tabla en Athena puede ejecutar queries y comprobar que Tableau puede halar los datos.

### Pasos previos

- Baja un archivo CSV que sea bien antiguo de la carpeta del reporte
	- En este caso de `therapist-forward-fill` en el data lake
- Escribe código que actualice ese CSV para que contenga la(s) columna(s) nueva(s)

> [!Warning]
> Antes de continuar: revisa si hay columnas que podrían faltar. Si las hay, el código debe tenerlas en cuenta para agregarlas.
>
> Se pueden agregar los espacios de cada fila vacíos.
> Se agrega de tal forma que un archivo viejo tenga la misma estructura que el más reciente.

- Conserva el archivo original descargado en algún archivo (GDrive o pregunta a Infra)
	- Comenta sobre esta copia en `#backend-prod-ops`
- Carga la nueva versión con la(s) nueva(s) columna(s)

### Para comprobar si funciona

- Corre el crawler
- Corre una query sobre la tabla modificada
	- En este caso `therapist-forward-fill`

> [!Important]
> En Omega: pide a Saumil que refresque Tableau para la tabla modificada y revise si hay algún error.
>
> En Alpha: como no hay Tableau solo puedo confirmarme con correr la query y ver que no se rompa.

### Si todo lo anterior va bien...

se aplica todo en general usando el AWS CLI

- Descarga una copia entera de la carpeta del reporte
- Haz una copia, comprímela y guárdala en algún archivo (GDrive o pregunta a Infra)
- Corre el script para agregar las columnas faltantes en la copia local

> [!Warning]
> Antes de correr el último comando, postea en `#backend-prod-ops`. No ejecutes sin la previa autorización y aprobación de Ryan.

- Corre `aws cp` de la carpeta local para cargar todos los CSVs a la subcarpeta en el data lake
	- En este caso `therapist-forward-fill`



### Cantidad de columnas con el export de TIR

Estas son las columnas:
```
snapshot_date
therapist_id
first_name
last_name
clinicient_staff_id
status
last_sign_in_date
last_session_date
is_accepting_new_patients
is_available_in_multiple_areas
powered_by_partner_code
origin_address
origin_zip_code
period_index
period_start
period_end
target_bookings
actual_bookings
available_minutes
available_minutes_net_time_off
treatment_area_available_minutes
treatment_area_available_minutes_net_time_off
accepting_new_patients
inactive_reason_key
inactive_reason_title
inactive_reason_notes
```

Total: 26 columnas.

## Estado de las subcarpetas en Alpha y Omega

### Tamaño de los CSV en Alpha

```bash
aws s3 ls s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/ --recursive --summarize --human-readable | grep "Total Size"
   
   Total Size: 56.5 GiB
```

### Tamaño de los CSV en Omega

```bash
aws s3 ls s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/ --recursive --summarize --human-readable | grep "Total Size"
   
   Total Size: 148.2 GiB
```

## Tamaño de las carpetas por año ⚖️

Dado los grandes tamaños que tienen ambas subcarpetas en el bucket, lo más eficiente para reducir el riesgo y poder responder mejor es bajar todo por año.

### Tamaño en 2023

Alpha:
```bash
aws s3api list-objects-v2 --bucket luna-alpha-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2023" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 17531825755 bytes (16.3278 GB)
```

### Tamaño en 2024

Alpha:
```bash
aws s3api list-objects-v2 --bucket luna-alpha-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2024" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 40068803509 bytes (37.317 GB)
```

### Tamaño en 2025

Alpha:
```bash
aws s3api list-objects-v2 --bucket luna-alpha-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2025" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 3047939019 bytes (2.83861 GB)
```

## Ejecutar el backfill 🛠️

> [!Tip]
> Para descargar es mejor usar `aws s3 sync` ya que permite resumir la descarga en caso de falla sin tener que bajar de nuevo todos los archivos.

> [!Warning]
> Recuerda comprimir la carpeta para conservar un archivo y tener forma de restaurar en caso de falla.

### Año 2023 de Alpha ✅

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-alpha-2023/ --exclude "*" --include "2023*"
```

Correr la rake:
```bash
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-alpha-2023]
```

Volver a cargar las carpetas del año 2023:
```bash
aws s3 sync ~/Downloads/tir-backfill-alpha-2023/ s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/
```

### Año 2024 de Alpha

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-alpha-2024/ --exclude "*" --include "2024*"
```

Correr la rake:
```bash
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-alpha-2024]
```

Volver a cargar las carpetas del año 2023:
```bash
aws s3 sync ~/Downloads/tir-backfill-alpha-2024/ s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/
```

