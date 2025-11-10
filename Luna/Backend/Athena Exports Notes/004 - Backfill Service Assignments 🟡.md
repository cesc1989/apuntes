# 014 - Backfill Service Assignments

Etiquetas: #luna_help_desk 

Caso EDG-2571

Reporte:
> Data goes back to 2020, but it's most important that January - May 2025 be updated.

## Contexto

Este export se da desde Grimoire. Lo que quieren es correr el worker para hacer backfill de los nuevos campos agregados a los archivos de años anteriores.

Los nuevos campos que se agregaron son:

- notes_left
- first_note_left
- last_note_left
- agent_email

## Revisión en Alpha

Bajé el reporte de Noviembre de 2025 y puedo ver esas columnas presentes en el CSV. En cambio el de Enero de 2020 no lo tiene.

Puedo probar los workers en alpha de grimoire para tener claridad de cómo usarlos para Omega.

# Prueba en Alpha

> [!Important]
> En alpha como tengo permisos puedo correr los workers por año. Sin embargo, para Omega me tocaría pedir una prod-op.
>
> Entonces o hago el script que corra en local con Claudio o armo un script para que corra para todos los años y pido la prod-op.


## Descarga todas las carpetas de Alpha

```bash
aws s3 sync \
  s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ \
  ~/Downloads/sda-backfill-alpha/ \
  --exclude "*" \
  --include "2020*" \
  --include "2021*" \
  --include "2022*" \
  --include "2023*" \
  --include "2024*" \
  --include "2025-01*" \
  --include "2025-02*" \
  --include "2025-03*" \
  --include "2025-04*" \
  --include "2025-05*"
```

## Backfill

### Opción 1: script local para generar los CSVs de Alpha

Se debe ejecutar desde la carpeta de Grimoire.

Comprobar que los CSV descargados les falte la columna con:
```bash
bash ../backfill-scripts/service-assignments/check_csv_headers.sh
```

Luego hay que exportar el ENV con los valores para conectar a la DB de Alpha:
```bash
source setup_omega_env.sh
```

Correr el script local
```bash
mix run backfill_assignments_local.exs
```

Verificar que los archivos generados tienen las columnas esperadas
```bash
bash ../backfill-scripts/service-assignments/check_csv_headers.sh ~/Downloads/sda-backfill-alpha-modded/
```

### Opción 2: correr los workers en Alpha

> [!Info]
> Con el comando `bin/grimoire remote` ingreso a una sesión de iex en el contenedor.

```erlang
dates = ["2020-01-01", "2020-02-01", "2020-03-01", "2020-04-01", "2020-05-01", "2020-06-01", "2020-07-01", "2020-08-01", "2020-09-01", "2020-10-01", "2020-11-01", "2020-12-01"]

Enum.each(dates, fn date ->
	IO.puts("Processing #{date}...")
	Grimoire.Reporting.Workers.ServiceDesk.AssignmentsWorker.perform(%Oban.Job{args: %{"date" => date}})
	IO.puts("Completed #{date}")
end)
```

## Cargar las carpetas a S3 Alpha

```bash
aws s3 sync \
  ~/Downloads/sda-backfill-alpha-modded/ \
  s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/
```

# Omega

> [!Warning]
> Recuerda comprimir la carpeta para conservar un archivo y tener forma de restaurar en caso de falla.

Descarga los archivos:
```bash
aws s3 sync \
  s3://luna-omega-workloads-data-lake/business-operations/service-desk/assignments/ \
  ~/Downloads/sda-backfill-omega/ \
  --exclude "*" \
  --include "2020*" \
  --include "2021*" \
  --include "2022*" \
  --include "2023*" \
  --include "2024*" \
  --include "2025-01*" \
  --include "2025-02*" \
  --include "2025-03*" \
  --include "2025-04*" \
  --include "2025-05*"
```

## Backfill en Local ✅

Comprobar que los CSV descargados les falte la columna con:
```bash
bash ../backfill-scripts/service-assignments/check_csv_headers.sh ~/Downloads/sda-backfill-omega/
```

Luego hay que exportar el ENV con los valores para conectar a la DB de Omega:
```bash
source setup_omega_env.sh
```

Correr el script local
```bash
mix run backfill_assignments_local.exs
```

Verificar que los archivos generados tienen las columnas esperadas
```bash
bash ../backfill-scripts/service-assignments/check_csv_headers.sh ~/Downloads/sda-backfill-omega-modded/
```

Compara los datos entre los originales y los modificados
```bash
bash ../backfill-scripts/service-assignments/compare_csv_data.sh ~/Downloads/sda-backfill-omega/ ~/Downloads/sda-backfill-omega-modded/
```

## Carga las carpetas a S3 🟡

> [!Warning]
> Antes de correr este último comando, postea en `#backend-prod-ops`. No ejecutes sin la previa autorización y aprobación de Ryan.

```bash
aws s3 sync \
  ~/Downloads/sda-backfill-omega-modded/ \
  s3://luna-omega-workloads-data-lake/business-operations/service-desk/assignments/
```

## Pasos Finales

- [x] Corre query en Athena antes de correr el Crawler
	- Descarga copia de los resultados
- [ ] Corre el crawler
- [ ] Vuelve a correr query en Athena
	- Descarga copia de los resultados y compara