# Apuntes de Use the Index, Luke de Markus Winand

Resaltos y comentarios en Hypothesis.

## Prefacio

El lenguaje SQL es quizás el más exitoso de entre los de cuarta generación (4GL). ==Su principal ventaja es su capacidad para separar _qué es_ y _quién es_. Una sentencia SQL es una descripción estructurada de _qué es_ lo que se necesita, sin tener que especificar instrucciones de _cómo_ puede ser ejecutado:==

```sql
SELECT date_of_birth
  FROM employees
 WHERE last_name = 'WINAND'
```

> Para escribir una sentencia SQL no se requiere ningún tipo de conocimiento a cerca del funcionamiento interno de la base de datos o del almacenamiento del sistema (como discos o archivos)

Esto en contra partido a la programación normal donde a veces hay que entender cosas como IO o el sistema de archivos.

> la única cosa que los _desarrolladores_ deben aprender es a indexar. La indexación de una base de datos es, de hecho, una tarea de programación

> La información más importante para indexar es como la aplicación selecciona los datos.

> el libro profundiza en un tipo de índices, el más importante de todos: el _índice B-tree_.


## Anatomía de un índice SQL

> Un índice es una estructura diferente dentro de la base de datos; creado con el comando `create index`. Requiere su propio espacio en disco y contiene una copia de los datos de la tabla. ==Eso significa que un índice es una redundancia.==

> un índice de base de datos se parece mucho a un índice de un libro: ocupa su propio espacio, es redundante y hace referencia a la información actual almacenada en otro lugar.

### El árbol de búsqueda (B-tree) hace el índice rápido

Capítulo: https://use-the-index-luke.com/es/sql/%C3%ADndice-anatom%C3%ADa/b-tree

> Una vez creado el índice, la base de datos lo mantiene automáticamente. Se aplican cada `insert`, `delete` y `update` al índice y se conserva el árbol equilibrado, lo que genera una sobrecarga de mantenimiento para las operaciones de escritura.


Sobre la profundidad del índice B-tree:
> Eso significa que la profundidad del árbol crece lentamente en comparación al número de hojas. Hay índices reales con millones de registros que tienen una profundidad de cuatro o cinco. Es poco común encontrar una profundidad de seis.

#### Recursos

- Simulador de árbol binario: https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html


### Índices lentos, Sección I

Capítulo: https://use-the-index-luke.com/es/sql/%C3%ADndice-anatom%C3%ADa/%C3%ADndices-lentos

> La segunda causa de búsquedas lentas aun usando un índice es tener que ir a la tabla.

> ==Una búsqueda por un índice requiere tres etapas: (1) el recorrido del árbol; (2) seguir la cadena de los nodos hojas; (3) devolver los datos de la tabla==. El recorrido del árbol es la única etapa que tiene acceso a un número limitado de bloques, corresponde a la profundidad del árbol. ==Las otras dos etapas deberían tener acceso a muchos bloques que pueden ser la causa de la lentitud durante una búsqueda a través de un índice.==


Sobre el mito de índice lento (ver: https://use-the-index-luke.com/es/sql/directorio-de-mitos/los-indices-pueden-degradarse)
> El origen del mito de los índices lentos es debido a la falsa creencia de que la búsqueda sólo recorre el índice, así que genera la idea de que el índice lento puede estar causado por un árbol “roto” o “desequilibrado”.


## El Filtro where

> Aunque el filtro `where` tiene un impacto importante sobre el rendimiento, generalmente se dice que eso se debe a que lee una gran parte del índice. ==La conclusión es que una mala programación del filtro `where` es el primer ingrediente de una sentencia lenta==.


### Operador de Igualdad

#### Llave primaria

Pone de ejemplo esta query:
```sql
SELECT first_name, last_name
  FROM employees
 WHERE employee_id = 123
```

Y este plan de ejecución (Oracle):
```
---------------------------------------------------------------
|Id |Operation                   | Name         | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT            |              |    1 |    2 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES    |    1 |    2 |
|*2 |  INDEX UNIQUE SCAN         | EMPLOYEES_PK |    1 |    1 |
---------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access("EMPLOYEE_ID"=123)
```

> ==Después de tener acceso al índice, la base de datos debe hacer una etapa más para devolver los datos seleccionados (`FIRST_NAME`, `LAST_NAME`) desde el almacenamiento de la tabla: la operación `TABLE ACCESS BY INDEX ROWID`==. Esta operación podría ser un cuello de botella como se ha explicado en "Índices lentos, Sección I" aunque aquí no existe ningún riesgo porque se realiza un `INDEX UNIQUE SCAN`. ==Esta operación no puede devolver más de un registro así que no podrá hacer más de un acceso a la tabla. Eso significa que los ingredientes para una sentencia lenta no se dan en la operación `INDEX UNIQUE SCAN`.==


#### Índices concatenados

> Aunque la base de datos crea de manera automática el índice sobre la llave primaria, puede ser útil, para una afinación manual, crear un índice sobre todo si está compuesto por varias columnas. En este caso, la base de datos crea un índice sobre todas las columnas de la llave primaria, que se llama _índice_ índice multi-columna (también conocido como índice _compuesto_ o índice _combinado_). Apunten que ==el orden de las columnas en los índices concatenados tiene un impacto positivo sobre su uso, así que se debe escoger con cuidado.==

Índice y query de ejemplo:
```sql
CREATE UNIQUE INDEX employees_pk
    ON employees (employee_id, subsidiary_id)
```

```sql
SELECT first_name, last_name
  FROM employees
 WHERE employee_id   = 123
   AND subsidiary_id = 30
```

> Cada vez que una sentencia utiliza la llave primaria completa, la base de datos puede usar un `INDEX UNIQUE SCAN` sin importar cuántas columnas tenga el índice. Pero, ==¿qué pasa cuando se usa solamente una de las columnas de la llave, por ejemplo, cuando se buscan todos los empleados de la llave secundaria?

```sql
SELECT first_name, last_name
  FROM employees
 WHERE subsidiary_id = 20
```

> ==El plan de ejecución revela que la base de datos no usó el índice. En su lugar, se empleó `TABLE ACCESS FULL`.==


> Un índice concatenado es solamente un índice B-tree como cualquier otro conservando los datos indexados dentro de una lista ordenada. La base de datos considera cada columna de acuerdo con su posición en la definición del índice para ordenar las entradas del mismo. ==La primera columna es el primer criterio de ordenamiento y la segunda columna determina el orden solamente si dos entradas tienen el mismo valor en la primera columna.==

> El orden de un índice con dos columnas es por lo tanto como el de una guía telefónica: está ordenado primero por apellidos, y después por nombres. ==Eso quiere decir que un índice con dos columnas no soporta una búsqueda únicamente sobre la segunda columna; es como si no se pudiera buscar por nombres dentro de una guía telefónica.==

> [!Important]
> La consideración más importante cuando se define un índice concatenado es cómo escoger el orden de las columnas, así que se deberá elegir la que puede ser más usada.

> Aunque la solución de los dos índices da muy buenos resultados para la operación `select`, es preferible la solución de un único índice. No es solamente para ahorrar espacio, sino también para el mantenimiento del segundo índice. **Cuantos menos índices tenga una tabla, mejor rendimiento darán las instrucciones `insert`, `delete` y `update`**.

Quienes deben crear los índices en la base de datos son los desarrolladores trabajando a diario con el sistema:
> Los desarrolladores tienen el conocimiento de los datos y conocen los caminos de acceso a los mismos. Son quienes pueden indexar correctamente para obtener el máximo beneficio de la aplicación (...).

Y por eso es pertinente tener claro cómo sacarle mejor provecho.

#### Índices Lentos, Parte II

Capítulo: https://use-the-index-luke.com/es/sql/where/operadores-de-igualdad/%C3%ADndices-lentos


> Cambiar un índice, sin embargo, podría afectar todas las sentencias de la tabla indexada.

##### Plan con un índice dedicado

índice:
```sql
CREATE INDEX emp_name ON employees (last_name)
```

Plan de ejecución:
```
--------------------------------------------------------------
| Id | Operation                   | Name      | Rows | Cost |
--------------------------------------------------------------
|  0 | SELECT STATEMENT            |           |    1 |    3 |
|* 1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES |    1 |    3 |
|* 2 |   INDEX RANGE SCAN          | EMP_NAME  |    1 |    1 |
--------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   1 - filter("SUBSIDIARY_ID"=30)
   2 - access("LAST_NAME"='WINAND')
```

> Un índice correctamente definido es aún mejor que el escaneo entero de la tabla propuesto originalmente.


> Usar un índice no implica necesariamente que la sentencia se ejecute de la mejor manera posible.

### Funciones

#### Sin distinción entre mayúscula y minúscula usando UPPER o LOWER

Esta query:
```sql
SELECT first_name, last_name, phone_number
  FROM employees
 WHERE UPPER(last_name) = UPPER('winand')
```

Da este plan de ejecución:
```
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |   10 |  477 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |   10 |  477 |
----------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   1 - filter(UPPER("LAST_NAME")='WINAND')
```

Hace un full table scan a pesar de tener un índice:
> Esto es el regreso de nuestro viejo amigo FULL TABLE SCAN. Aunque existe un índice sobre la columna `LAST_NAME`, es inutilizable -porque la búsqueda _no_ es sobre `LAST_NAME` sino sobre `UPPER(LAST_NAME)`. ==Desde el punto de vista de la base de datos, es algo _completamente diferente_.==


> Es una trampa en la cual es fácil caer. Reconocemos al instante la relación entre `LAST_NAME` y `UPPER(LAST_NAME)` y esperemos que la base de datos “vea” lo mismo. En realidad, la visión del optimizador es más como esto otro:

```sql
SELECT first_name, last_name, phone_number
  FROM employees
 WHERE BLACKBOX(...) = 'WINAND'
```

> La función `UPPER` es solamente una caja negra. ==Los parámetros hacia la función no son pertinentes porque no existe una relación general entre los parámetros de las funciones y el resultado.==


> Para soportar esta sentencia, se requiere un índice adicional para cubrir la expresión de la búsqueda actual. ==Eso significa que no se necesita un índice sobre `LAST_NAME` sino uno sobre `UPPER(LAST_NAME)`:==

```sql
CREATE INDEX emp_up_name 
    ON employees (UPPER(last_name))
```


> [!Note]
> SQL Server y MySQL no soportan los índices basados en funciones como se ha descrito pero ambos dan la posibilidad de usar, en su lugar, las columnas virtuales. Para utilizarlas, se debe agregar primero una columna virtual a la tabla para poder indexarla más tarde.


#### Funciones definidas por usuario

Capítulo: https://use-the-index-luke.com/es/sql/where/funciones/definidas-por-el-usuario

Aclara que solo las funciones deterministas pueden ser indexadas. Sino son cajas negras para el optimizador del motor de la base de datos:
> Solamente las funciones que devuelven siempre el mismo resultado con los mismos parámetros pueden ser indexadas.


### Sentencias con variables

El capítulo me pareció algo confuso así que le pedí un resumen al asistente de Kagi. Esto dijo:

> Las variables bind permiten reutilizar el plan de ejecución, así que la base de datos no reconstruye el plan cada vez que cambia un valor. Eso ahorra recursos .
>
> **El problema** es que, al no conocer el valor concreto, ==el optimizador asume que los datos están distribuidos de forma uniforme. Siempre calcula el mismo número estimado de registros y elige el mismo plan==, aunque con ciertos valores un índice sería genial y con otros sería una mala opción .
>
> **La clave del artículo:** usalas por defecto, excepto cuando el valor concreto sí pueda cambiar el plan de ejecución (p.ej., valores muy comunes vs. muy raros). En esos casos puntuales, un literal es mejor

Resumen: bind por defecto, literal solo cuando el valor real importe para el índice.

> [!Important]
> Esto aclara el capítulo sobre lo de "valores muy comunes vs muy raros":
> > Consideremos por ejemplo los valores para los estados “hecho” y “por hacer”, que típicamente están distribuidos de manera desigual. El número de entradas “hecho” excede generalmente los registros “por hacer” por una gran magnitud. Usar un índice tiene sentido sólo cuando se busca entre las entradas “por hacer”.

### Búsqueda por rangos

#### Mayor que, menor que y BETWEEN

tbc

#### Indexar Filtros LIKE SQL

> El operador SQL `LIKE` puede dar lugar a un comportamiento inesperado en el rendimiento porque algunos criterios de búsqueda previenen de manera eficiente el uso del índice. ==Eso significa que existen criterios de búsqueda que pueden ser muy bien indexados, pero otros no lo permiten. Es la posición del comodín el cual provoca la diferencia.==

> [!Important]
> Evitar las expresiones `LIKE` con el comodín como primer renglón (p.ej., `'%TERM'`).


> Para la base de datos PostgreSQL, el problema es diferente porque Post­greSQL asume que _existe_ un comodín como primer renglón cuando se usa una variable Bind con una expresión `LIKE`. En este caso, PostgreSQL no podrá usar índices.

#### Índice Combinado

> ¿es mejor crear un índice por cada columna o un único índice con todas las columnas del filtro `where`? ==La respuesta es muy sencilla en muchos casos: un índice con múltiples columnas es mejor, es un índice concatenado o compuesto.==


> Los data warehouses usan un tipo de índice especial para resolver ese problema: el llamado _indice bitmap_. La ventaja de los índices bitmap es que es posible mezclarlos con bastante facilidad. Eso significa que obtienes un rendimiento aceptable cuando se indexa cada columna de manera individual.


### Índices Parciales

> Un índice parcial es muy útil para los filtros `where` comúnmente usados, que usan valores constantes, como un código de estado en el siguiente ejemplo:

```sql
SELECT message
  FROM messages
 WHERE processed = 'N'
   AND receiver  = ?
```

> Con un índice parcial, se puede limitar el índice para incluir solamente los mensajes no procesados. La sintaxis para este índice es sorprendentemente sencilla: un filtro `where`.

```sql
CREATE INDEX messages_todo
          ON messages (receiver)
       WHERE processed = 'N'
```


> El filtro `where` de un índice parcial puede llegar a ser arbitrariamente complejo. ==La única limitación esencial está en las funciones: se pueden usar solamente las funciones deterministas, al igual que sucede en las otras partes de la definición del índice.==


### NULL en la base de datos Oracle

> El estándar SQL no define `NULL` como un valor, sino más bien como un marcador de posición para un valor ausente o desconocido. Por consiguiente, ningún valor puede ser `NULL`.


#### NULL dentro de los índices

> Se puede extender este concepto para la sentencia original con el fin de encontrar todos los registros donde `DATE_OF_BIRTH` `IS NULL`. ==Para eso, la columna `DATE_OF_BIRTH` tiene que ser la que esté más a la izquierda dentro del índice, así puede ser usada como predicado de acceso. Aunque no se necesita el segundo índice para esta sentencia, agregamos otra columna que nunca podrá ser `NULL` para asegurarnos de que el índice tenga todos los registros.==


> [!Important]
> Agregar una columna que no puede ser `NULL` para indexar `NULL` como un valor.


#### Restricciones NOT NULL

> Para indexar una condición `IS NULL` en la base de datos Oracle, el índice debe tener una columna que nunca ha de ser `NULL`.

> [!Note]
> Una restricción `NOT NULL` eliminada puede prevenir el uso de un índice dentro de la base de datos.


#### Emulando índices parciales en la base de datos Oracle

tbc


### Condiciones complicadas

#### Fechas

Esto:
```sql
SELECT ...
  FROM sales
 WHERE TRUNC(sale_date) = TRUNC(sysdate - INTERVAL '1' DAY)
```

Es una sentencia perfectamente válida y correcta pero no se puede usar correctamente el índice sobre `SALE_DATE`. Es como lo explicado en la sección [“_Sin distinción entre mayúscula y minúscula usando `UPPER` o `LOWER`_”](https://use-the-index-luke.com/es/sql/where/funciones/case-insensitive); ==`TRUNC(sale_date)` es algo completamente diferente de `SALE_DATE`. Las funciones son una caja negra para la base de datos.==

> Existe una solución bastante sencilla para este problema: un [índices sobre expresiones](https://use-the-index-luke.com/es/sql/where/funciones).

```sql
CREATE INDEX index_name
          ON sales (TRUNC(sale_date))
```

...

> La alternativa es usar una condición de rango explícita. Esta solución genérica funciona con todas las bases de datos:

```sql
SELECT ...
  FROM sales
 WHERE sale_date BETWEEN quarter_begin(?) 
                     AND quarter_end(?)
```


#### Números

tbc

#### Columnas combinadas

tbc

#### Lógica inteligente

tbc

#### Matemáticas

tbc


### La operación de unión (join)

> Aunque el orden de la unión no tiene impacto sobre el resultado final, afecta al rendimiento. Por lo tanto, el optimizador evaluará todas las combinaciones de orden posibles de la unión y seleccionará la mejor. Eso significa que el mero hecho de optimizar una sentencia compleja podría convertirse en un problema de rendimiento. Cuantas más tablas tenga la unión, más variantes de planes de ejecución hay que evaluar.

#### Loops anidados

tbc

#### Hash join

tbc

#### Soft-merge join

tbc

### Agrupación de datos

#### Filtros de predicados usados intencionalmente sobre índices

...

> Para aplicar este concepto sobre la sentencia anterior, se debe extender el índice para cubrir todas las columnas del filtro `where`, incluso aunque el rango escaneado del índice no se reduzca:

```sql
CREATE INDEX empsubupnam ON employees
       (subsidiary_id, UPPER(last_name))
```

> La columna `SUBSIDIARY_ID` es la primera columna del índice así que se puede usar como predicado de acceso. La expresión `UPPER(last_name)` cubre el filtro `LIKE` como _predicado de filtro del índice_. Indexando los datos con mayúsculas ahorraría algunos ciclos de CPU durante la ejecución, pero un índice “directo” sobre `LAST_NAME` funcionaría también bien. Se explicará con más detalle en la siguiente sección.


> [!Important]
> No se debe introducir un nuevo índice con el único propósito de tener predicados de filtro. En su lugar, extender un índice existente para conservar el [esfuerzo de mantenimiento](https://use-the-index-luke.com/es/sql/dml) bajo. Incluso con algunas bases de datos, se deben agregar columnas al índice de la clave primaria que no son parte de la clave primaria.


#### Escaneado limitado al índice: Evitar el acceso a la tabla

> Para cubrir una sentencia completa, un índice debe contener _todas_ las columnas de la sentencia SQL; especialmente las columnas de la cláusula `select` como se muestra en el siguiente ejemplo:

```sql
CREATE INDEX sales_sub_eur
    ON sales
     ( subsidiary_id, eur_value )
```

```sql
SELECT SUM(eur_value)
  FROM sales
 WHERE subsidiary_id = ?
```

Por supuesto, indexar el filtro `where` prevalece sobre las otras cláusulas. Por lo tanto, la columna `SUBSIDIARY_ID` está en primera posición así que cumple los requisitos para realizar un predicado de acceso.

> El escaneo de sólo índice es una estrategia de indexación agresiva. No se diseña un índice para “sólo escanear el índice” únicamente “por si acaso”, porque sino se usaría sin necesidad la memoria y se incrementaría el esfuerzo de mantenimiento necesario para las sentencias `update`.


> El acceso a la tabla incrementa el tiempo de respuesta a pesar de que la sentencia selecciona pocas filas. El factor pertinente no es cuántas filas devolverá la sentencia, sino cuántas filas deberá examinar la base de datos para encontrar lo que se busca.

> Combinar las sentencias `LIKE` en una sola, como se ha visto anteriormente, genera buenos candidatos para el escaneado limitado al índice. De este modo, se consultan muchas filas pero pocas columnas, lo que genera un índice pequeño y suficiente para soportar esta técnica. Cuantas más columnas se consultan, más columnas se tienen que agregar al índice para soportar el escaneado de sólo índice índice. Como desarrollador, se deben seleccionar solamente las columnas que sean realmente necesarias.


> [!Note]
> Evitar `select *` y buscar solamente las columnas que se necesitan.


> Indexar muchas filas requiere mucho espacio y, además, se puede alcanzar el límite de la base de datos. La mayoría de las bases de datos imponen un límite estático para el número de columnas dentro de un índice y para el tamaño total de entradas dentro de un índice. Eso significa que no pueden indexar un número caprichoso de columnas ni columnas de tamaño arbitrario.


#### Índices Organizados en Tablas (IOT)

tbc

### Ordenar y Agrupar



### Resultados Parciales



### Inser, Delete, Update