# Notas sobre Logical Replication

Logical Replication (LR) es el proceso de sincronizar tablas de diferentes bases de datos de una manera rápida y con poca incidencia de errores. Se logra al crear la tabla a duplicar en la base de datos objetivo siguiendo una serie de prerequisitos. Una vez los requisitos están en lugar la sincronización de datos ocurre de manera rápida y con mínimos errores.

Para completar AppBlend M3 se va usar LR para sincronizar la base de datos de Patient Forms y de Clinical Dashboard en Edge.

Lo descrito a continuación aplica a bases de datos PostgreSQL, generalmente.

>[!Info]
>
> S = Source table
> T = Target table
>
> Toda columna en S tiene que estar en T.
> T puede tener más columnas que S.

## Condiciones

- Las tablas deben estar en el mismo schema
- Las tablas deben tener el mismo nombre en S y en T
- Los nombres de columnas deben ser el mismo en S y en T
- Los tipos de dato deben ser los mismos en S y en T

## Convención de schema en T

- Todas las columnas deben ser nuleables
	- Excepto la llave primaria
- Todas las llaves foráneas deben ser agregadas
- Todos los índices en S no deben existir en T
	- Excepción de la llave primaria
	- Los índices se pueden recrear después que se configuré LR

## Ejemplo de Edge -> Grimoire

Edge - Source table

![[01.lr.example.source.png]]

Grimoire - Target table

![[02.lr.example.target.png]]

### Detalles

- Todas las columnas en T son nuleables
- Se quitaron los índices en T
- Hay llaves foráneas en T
- No hay secuencia en la llave primaria en T
- Ninguna columna en T tiene valores por defecto
- Todos los nombres coinciden: tabla, campos, tipos de datos
- Orden de las columnas coincide. Esto es un plus.