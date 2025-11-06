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

# Alpha Backfill para Probar

> [!Important]
> En alpha como tengo permisos puedo correr los workers por año. Sin embargo, para Omega me tocaría pedir una prod-op.
>
> Entonces o hago el script que corra en local con Claudio o armo un script para que corra para todos los años y pido la prod-op.

## 2025

Teniendo en cuenta que Brett pide prioridad para Enero - Mayo de 2025, empezaré por estos.

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ ~/Downloads/sda-backfill-alpha-2020/ --exclude "*" --include "2025-01*" --include "2025-02*" --include "2025-03*" --include "2025-04*" --include "2025-05*"
```

## 2020

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ ~/Downloads/sda-backfill-alpha-2020/ --exclude "*" --include "2020*"
```


Comando para correr los workers en Alpha:
```erlang
dates = ["2020-01-01", "2020-02-01", "2020-03-01", "2020-04-01", "2020-05-01", "2020-06-01", "2020-07-01", "2020-08-01", "2020-09-01", "2020-10-01", "2020-11-01", "2020-12-01"]

Enum.each(dates, fn date ->
	IO.puts("Processing #{date}...")
	Grimoire.Reporting.Workers.ServiceDesk.AssignmentsWorker.perform(%Oban.Job{args: %{"date" => date}})
	IO.puts("Completed #{date}")
end)
```


## 2021

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ ~/Downloads/sda-backfill-alpha-2021/ --exclude "*" --include "2021*"
```

## 2022

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ ~/Downloads/sda-backfill-alpha-2022/ --exclude "*" --include "2022*"
```

## 2023

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ ~/Downloads/sda-backfill-alpha-2023/ --exclude "*" --include "2023*"
```

## 2024

Descarga las carpetas:
```bash
aws s3 sync s3://luna-alpha-workloads-data-lake/business-operations/service-desk/assignments/ ~/Downloads/sda-backfill-alpha-2024/ --exclude "*" --include "2024*"
```