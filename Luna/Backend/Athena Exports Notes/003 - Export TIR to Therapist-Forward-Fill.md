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
and inactive_reason_notes <> ''
limit 100;
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

```sql
SELECT *
FROM "business-operations"."therapist_forward_fill" tfi
where tfi.clinicient_staff_id != 'UNKNOWN'
limit 20;
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

### Tamaño en 2020

Omega:
```bash
aws s3api list-objects-v2 --bucket luna-omega-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2020" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {pr
int "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 7219931201 bytes (6.72408 GB)
```

### Tamaño en 2021

Omega:
```bash
aws s3api list-objects-v2 --bucket luna-omega-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2021" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 17887166294 bytes (16.6587 GB)
```

### Tamaño en 2022

Omega:
```bash
aws s3api list-objects-v2 --bucket luna-omega-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2022" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 36066670455 bytes (33.5897 GB)
```

### Tamaño en 2023

Alpha:
```bash
aws s3api list-objects-v2 --bucket luna-alpha-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2023" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 17531825755 bytes (16.3278 GB)
```

Omega:
```bash
aws s3api list-objects-v2 --bucket luna-omega-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2023" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {pr
int "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 60099717992 bytes (55.9722 GB)
```

### Tamaño en 2024

Alpha:
```bash
aws s3api list-objects-v2 --bucket luna-alpha-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2024" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 40068803509 bytes (37.317 GB)
```

Omega:
```bash
aws s3api list-objects-v2 --bucket luna-omega-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2024" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {pr
int "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 36047098952 bytes (33.5715 GB)
```

### Tamaño en 2025

Alpha:
```bash
aws s3api list-objects-v2 --bucket luna-alpha-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2025" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {print "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 3047939019 bytes (2.83861 GB)
```

Omega:
```bash
aws s3api list-objects-v2 --bucket luna-omega-workloads-data-lake --prefix "business-operations/therapist-forward-fill/2025" --query "Contents[].Size" --output text | tr '\t' '\n' | awk '{sum += $1} END {pr
int "Total size:", sum, "bytes (" sum/1024/1024/1024 " GB)"}'

Total size: 1785894140 bytes (1.66324 GB)
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

### Año 2024 de Alpha ✅

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-alpha-2024/ --exclude "*" --include "2024*"
```

Correr la rake:
```bash
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-alpha-2024]
```

Volver a cargar las carpetas del año 2024:
```bash
aws s3 sync ~/Downloads/tir-backfill-alpha-2024/ s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/
```

### Año 2025 de Alpha ✅

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-alpha-2025/ --exclude "*" --include "2025*"
```

Correr la rake:
```bash
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-alpha-2025]
```

Volver a cargar las carpetas del año 2025:
```bash
aws s3 sync ~/Downloads/tir-backfill-alpha-2025/ s3://luna-alpha-workloads-data-lake/business-operations/therapist-forward-fill/
```

## Backfill Definitivo en Omega

> [!Warning]
> Recuerda comprimir la carpeta para conservar un archivo y tener forma de restaurar en caso de falla.

> [!Warning]
> Antes de correr el último comando, postea en `#backend-prod-ops`. No ejecutes sin la previa autorización y aprobación de Ryan.

### Año 2020 en Omega ✅

Descarga las carpetas ✅
```bash
aws s3 sync s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-omega-2020/ --exclude "*" --include "2020*"
```

Correr la rake ✅
```
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-omega-2020]
```

Volver a cargar las carpetas del año 2020 ✅
```bash
aws s3 sync ~/Downloads/tir-backfill-omega-2020/ s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/
```

### Año 2021 en Omega ✅

Descarga las carpetas ✅
```bash
aws s3 sync s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-omega-2021/ --exclude "*" --include "2021*"
```

Correr la rake ✅
```
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-omega-2021]
```

Volver a cargar las carpetas del año 2021 ✅
```bash
aws s3 sync ~/Downloads/tir-backfill-omega-2021/ s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/
```

### Año 2022 en Omega ✅

Descarga las carpetas ✅
```bash
aws s3 sync s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-omega-2022/ --exclude "*" --include "2022*"
```

Correr la rake ✅
```
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-omega-2022]
```

Volver a cargar las carpetas del año 2022 ✅
```bash
aws s3 sync ~/Downloads/tir-backfill-omega-2022/ s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/
```

### Año 2023 en Omega

Descarga las carpetas ✅
```bash
aws s3 sync s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/ ~/Downloads/tir-backfill-omega-2023/ --exclude "*" --include "2023*"
```

Correr la rake ✅
```
bundle exec rake therapist_forward_fill:backfill_columns[/Users/francisco/Downloads/tir-backfill-omega-2023]
```

Volver a cargar las carpetas del año 2023
```bash
aws s3 sync ~/Downloads/tir-backfill-omega-2023/ s3://luna-omega-workloads-data-lake/business-operations/therapist-forward-fill/
```

### Año 2024 en Omega


### Año 2025 en Omega

# Errores ❌

Las cosas que pueden fallar al hacer el backfill y cómo solventar.

## Crawler crea tablas por cada carpeta

> [!Important]
> Este error se da por no actualizar el Classifier con las columnas nuevas. Para evitarlo hay que actualizar antes el Classifier con las columnas nuevas y luego correr el Crawler.

Cuando el Crawler detecta muchas particiones con diferentes columnas creará tablas por cada partición.

Se puede detectar este error cuando el Crawler en vez de actualizar particiones agrega nuevas tablas.

![[102.tir-backfill-2021-f2.png]]

Nota la ejecución del 24 de Septiembre a las 14:49. Agregó 71 mil tablas pero actualizó cero particiones. Eso fue el error.

Una vez actualicé el Classifier y corrí de nuevo el Crawler se puede ver el cambio en la ejecución del mismo día pero más tarde. Cambiaron 71 mil tablas (se marcaron para ser borradas) y se actualizaron 17 mil particiones.

Enlaces:

- [Multiple tables created after crawling data using glue from a s3 bucket](https://www.reddit.com/r/aws/comments/1gstb6y/multiple_tables_created_after_crawling_data_using/): un comentario da esa explicación.

## Borrar tablas creadas por error

Luego de lo anterior pasó que Glue marcó las tablas malas como _deprecated_. Les puso un timestamp. Pensé que eso significaría que Glue las iba a borrar pero no fue así.

La UI solo podía cargar 1000 tablas. En lugar de ir borrando solo cambiaba las listas.

Si seleccionaba alguna e intentaba borrar daba error por "uso concurrente" o "no encontrado". Lo que me hacía creer que se estaban borrando.

Error.

Había que borrar manualmente desde la UI o mediante el CLI. Entonces fue con el uso del CLI que listé todas las tablas y aún aparecían todas. Así que tocaba borrar también con el CLI.

Con esta comando listé todas las tablas y las guardé en un archivo de texto plano:
```bash
aws glue get-tables \
  --database-name business-operations \
  --query 'TableList[].Name' \
  --output json > tablas.txt
```

Luego intenté con este traer si estaban o no borradas pero no hay tal atributo:
```bash
aws glue get-tables \
  --database-name business-operations \
  --query 'TableList[?DeleteTime != null].[Name, DeleteTime]' \
  --output json > deleted_tables.txt
```

Las tablas que se podían borrar son las marcadas con un timestamp en _deprecated_.

Eso lo pude comprobar cuando al traer solo los datos de un par:
```bash
aws glue get-tables \
  --database-name business-operations \
  --expression "2020_04_17t23_00_07z|2020_04_17t23_30_07z" \
  --output json > specific_tables.txt
```

Pude revisar sus propiedades:
```json
{
	"Name": "2020_04_17t23_00_07z",
	"DatabaseName": "business-operations",
	"Owner": "owner",
	"CreateTime": "2025-09-24T09:29:11-05:00",
	"UpdateTime": "2025-09-24T10:52:07-05:00",
	"LastAccessTime": "2025-09-24T10:52:07-05:00",
	"Retention": 0,
	"StorageDescriptor": {
			"Columns": [],
			"BucketColumns": [],
			"SortColumns": [],
			"Parameters": {
					"skip.header.line.count": "1",
					"sizeKey": "467024",
					"objectCount": "1",
					"UPDATED_BY_CRAWLER": "Business Operations - Therapist Forward Fill",
					"DEPRECATED_BY_CRAWLER": "1758729127916",
			},
			"StoredAsSubDirectories": false
	},
	"PartitionKeys": [],
	"TableType": "EXTERNAL_TABLE",
	"Parameters": {
			"skip.header.line.count": "1",
			"sizeKey": "467024",
			"objectCount": "1",
			"UPDATED_BY_CRAWLER": "Business Operations - Therapist Forward Fill",
			"DEPRECATED_BY_CRAWLER": "1758729127916",
	},
	"CreatedBy": "arn:aws:sts::349935666759:assumed-role/AWSGlueServiceRole-Default/AWS-Crawler",
	"IsRegisteredWithLakeFormation": false,
	"CatalogId": "349935666759",
	"VersionId": "1",
	"IsMultiDialectView": false
}
```

Así que la alternativa que quedaba era con el CLI.

### Solución: borrar con el CLI

Finalmente, con ayuda de Claudio armé un script para agrupar los nombres de a 100 e ir corriendo el comando en baches para borrar estas tablas.

Primero probé el comando para borrar de a una tabla a la vez:
```bash
aws glue delete-table --database-name business-operations --name "2020_04_17t23_00_07z"
```

Luego el comando para borrar en baches:
```bash
aws glue batch-delete-table \
    --database-name business-operations \
    --tables-to-delete "2020_04_18t02_00_08z" "2020_04_18t02_30_05z" "2020_04_18t03_00_07z" "2020_04_18t03_30_04z" "2020_04_18t04_00_05z" "2020_04_18t04_30_06z" "2020_04_18t05_00_00z" "2020_04_18t05_30_07z" "2020_04_18t06_00_06z" "2020_04_18t06_30_05z" "2020_04_18t07_00_07z" "2020_04_18t07_30_06z" "2020_04_18t08_00_30z" "2020_04_18t08_30_05z" "2020_04_18t09_00_07z" "2020_04_18t09_30_06z" "2020_04_18t10_00_06z" "2020_04_18t10_30_06z" "2020_04_18t11_00_07z" "2020_04_18t11_30_07z" "2020_04_18t12_00_06z"
```

Este tiene la limitante que solo admite listas de 100 nombres para borrar. Fue ahí entonces cuando con Claudio armé el script, corrí el modo dry-run y después puse en ejecución para eliminar.