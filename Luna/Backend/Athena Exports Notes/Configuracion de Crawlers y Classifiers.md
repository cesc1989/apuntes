# Configuración de Crawlers y Classifiers en Glue

Etiquetas: #luna_help_desk 

Al exportar los datos al data lake, la forma en que permitimos el acceso a estos es a través de una tabla que se configura en AWS Glue. En este servicio se puede crear la tabla manualmente, indicando el origin de los datos. Esto tiene el limitante que cada vez que haya una actualización hay que ir a la configuración de la tabla y agregar los nuevos campos.

Para automatizar todo nos podemos valer de los Crawlers y Classifiers que ofrece Glue.

El Crawler es lo que se usa para revisar un origin de datos (el CSV en S3) y poner los datos disponibles en una tabla. Para lograrlo los crawlers corren los Classifiers. Estos son la forma en que un crawler puede interpretar los datos en el origen de datos.

Así que al crear un Crawler es conveniente definir un Classifier para ayudarle a entender lo que estamos exportando.

Enlaces

- [Glue Crawlers](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html)
- [Custom Classifiers](https://docs.aws.amazon.com/glue/latest/dg/custom-classifier.html)

## Patient Summary

Para esta tabla se configuró el Crawler de nombre "Business Operations - Patient Summary". Queda así:

![[003.bo.crawler.png]]

Junto con este crawler se creó el Classifier de mismo nombre;

![[004.bo.classifier.png]]

## Campos Contains header y Header

El archivo CSV que exportamos tiene sus respectiva cabeceras que corresponden a los nombres de los campos que queremos en la tabla. Para indicarle al Crawler cuales son esos nombres de columna configuramos los campos "Contains header" y "Header" en la creación del Classifier.

El Classifier "Business Operations - Patient Summary" define estos dos campos así:

- Contains header: Has headings
- Header: lista de los campos exportados separados por coma.

> [!Note]
> Contains header tiene tres opciones:
> - Detect headings
> - Has headings
> - No headings

Cuando se actualiza el export (CSV) hay que ir al classifier y agregar los campos nuevos.

> [!warning]
> El Crawler es capaz de actualizar las columnas de la tabla sin necesidad de actualizar la lista en el Classifier.

## Tablas

Toda esta configuración la hacemos para poder acceder esos datos mediante tablas en Athena. Las tablas son la interfaz con la que podemos hacer consultas a los datos.\

Entonces podríamos ver todo esto como un flujo que empieza en S3 (data lake) y termina en la tabla en Glue.

S3 -> Crawler + Classifier -> Tabla -> Athena query

Cuando exportamos nuevos datos hay que actualizar el Classifier y luego correr el Crawler para que entienda los datos y los exponga a la tabla.

Si el export agrega una nueva columna, al hacer lo anterior y revisar la tabla debo poder ver la nueva columna en la descripción de la tabla.