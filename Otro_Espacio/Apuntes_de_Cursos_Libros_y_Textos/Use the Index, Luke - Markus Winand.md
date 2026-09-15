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