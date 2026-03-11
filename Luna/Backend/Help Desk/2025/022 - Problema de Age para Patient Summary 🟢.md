# 022 - Error de campo Age en Athena para Patient Summary - BO

Etiquetas #luna_help_desk 

Caso EDG-3090

## Contexto

Este error en Athena:
> Error parsing column 'age' with value '' with error: 'java.lang.NumberFormatException: empty String'. Please remove or correct the data in this file: *s3://luna-omega-workloads-data-lake/business-operations/patient-summary/2022-06/data.csv*. To treat invalid data as null to avoid this error, add the table property 'use.null.for.invalid.data' = 'true' to the table containing the invalid data

El worker que exporta estos datos es `app/workers/athena/patient_summary_writer_worker.rb`.

## Problema

El problema está en que el paciente inicialmente reportado no tiene fecha de nacimiento en la base de datos:
```
1b29a92f-4bdf-4e4f-8228-b521ba388f51|Thomas|Jagger|male|||
```

La corrección que ofrece Claudio es poner un -1 en esos casos. Probé cambiando manualmente en el archivo reportado, subí a S3, corrí el crawler y luego obtuve el mismo error pero en otro archivo.

> BAD_DATA: Error parsing column 'age' with value '' with error: 'java.lang.NumberFormatException: empty String'. Please remove or correct the data in this file: *s3://luna-omega-workloads-data-lake/business-operations/patient-summary/2022-12/data.csv*. To treat invalid data as null to avoid this error, add the table property 'use.null.for.invalid.data' = 'true' to the table containing the invalid data.


Así que procedí a pedir varias cosas a Claudio. Previamente, corrí esta query en Omega para encontrar los pacientes que no tenían ni fecha de nacimiento ni estaban blacklisted:
```sql
select 
 a.id,
 a.first_name,
 a.last_name,
 a.birthday,
 a.blacklisted,
 a.created_at
from accounts a
where a.birthday is null
and a.blacklisted = false
order by a.created_at desc
;
```

Aparecieron 12 resultados con nombres que parecían de pacientes de prueba. Ya con esa lista le pedí a Claudio que:

- hiciera un script bash para armar el comando `aws s3 cp` para bajar los archivos del bucket según los años `created_at` de cada paciente de la lista
- después le pedí que actualizar el script `find_null_birthdays.sqh` para iterar en cada carpeta descargada de S3 y listar los ids de cada fila donde hubiera un registro sin fecha de cumpleaños
	- los ids los escribió a un archivo de texto plano
- con esos ids le pedí que hiciera otro script donde arreglara el CSV para cada uno
	- Claudio hizo que el script guardara un respaldo del archivo original
	- La salida era los comandos `aws s3 cp` para cargar cada archivo donde correspondía

Con todo lo anterior, verifiqué que los archivos actualizados fueran correctos, cargué a s3 y luego corrí el crawler de la tabla. Cuando el crawler finalizó, corrí la query de nuevo y no hubo error.