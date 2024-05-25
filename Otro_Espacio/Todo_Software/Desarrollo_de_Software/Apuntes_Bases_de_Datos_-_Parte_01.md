# Apuntes Bases de Datos - Parte 01

# PostgreSQL
- Upgrade postgresql from 9.3 to 9.4: [Medium](https://medium.com/@tk512/upgrading-postgresql-from-9-3-to-9-4-on-ubuntu-14-04-lts-2b4ddcd26535) - Ojo con los locales. Mal configurados impiden completar el tutorial
- `[psql: FATAL: database “<user>” does not exist](https://stackoverflow.com/questions/17633422/psql-fatal-database-user-does-not-exist)`: create the database for the user/role
- Verificar servicio postgres está corriendo en Ubuntu: `sudo service postgresql status`


## Cambiar contraseña de usuario *postgres*

Forma que debería ser:

    sudo -u postgres psql postgres
    
    # \password postgres
    
    Enter new password: 

Recursos:

- comandos postgresql: [Cheat Sheets PostgreSQL section](https://github.com/cesc1989/CheatSheets/tree/master/postgresql)
- configurar usuario postgresql: [CheatSheets](https://github.com/cesc1989/CheatSheets/tree/master/postgresql/queries#create-new-user)
- [SO](https://stackoverflow.com/questions/10845998/i-forgot-the-password-i-entered-during-postgres-installation) - [Serverfault](https://serverfault.com/questions/110154/whats-the-default-superuser-username-password-for-postgres-after-a-new-install)
## Sumar resultado de `count` postgresql

Así

    SELECT ( SELECT COUNT(*) FROM comments ) 
         + ( SELECT COUNT(*) FROM tags ) 
         + ( SELECT COUNT(*) FROM search )

Visto en [Stack Overflow](https://stackoverflow.com/questions/1160512/add-results-from-several-count-queries/1160538#1160538)

## Recover backup using `pg_restore` when it errs with `psql` (invalid position stuff)

`psql dash --host localhost --username franciscoquintero < ~/projects/dash/latest.dump.1`

> The input is a PostgreSQL custom-format dump.
> Use the pg_restore command-line client to restore this dump to a database.

The message is pretty straightforward.


## Cláusula SELECT en una WITH
> Estas son las common table expressions.

Vi esto en el proyecto Edge para exportar los datos a Athena:

    WITH real_patients
        AS (...),
    care_plan_summary
      AS (
        SELECT cp.id
          ,COUNT(DISTINCT (
              CASE
                WHEN a.state IN (
                    1
                    ,2
                    )
                  THEN a.id
                END
              )) AS completed_visits_count
          ,COUNT(DISTINCT (
              CASE
                WHEN a.state = 0 AND a.scheduled_date >= \'#{24.hours.ago.iso8601}\'
                  THEN a.id
                END
              )) AS pending_visits_count
        FROM latest_care_plan cp
        LEFT JOIN real_appointments a ON a.episode_id = cp.id
        GROUP BY cp.id
        )

Y de esa forma para leerlo está bien enredado pero en resumen se está usando la cláusula [SELECT en una WITH](https://www.postgresql.org/docs/9.1/queries-with.html).


> The basic value of SELECT in WITH is to break down complicated queries into simpler parts


# MySQL
## MySQL no tiene un tipo de campo *boolean*. Usa *tinyint*

[SO](https://stackoverflow.com/questions/9382508/mysql-workbench-trying-to-create-a-boolean-field-for-a-table)

> There is no such thing as a 'boolean' in MySql unfortunately.

[SO 2](https://stackoverflow.com/questions/11167793/boolean-or-tinyint-confusion)

> MySQL does not have internal boolean data type. It uses the smallest integer data type - TINYINT.
> 
> The BOOLEAN and BOOL are equivalents of TINYINT(1), because they are synonyms.

[SO 3](https://stackoverflow.com/a/289759/1407371)

> For MySQL 5.0.3 and higher, you can use `BIT`
> 
> Otherwise, according to the MySQL manual you can use `BOOL` or `BOOLEAN`, which are at the moment aliases of [tinyint](http://dev.mysql.com/doc/refman/5.7/en/integer-types.html)(1)


## Diferencias entre las líneas de las relaciones en MySQL Workbench
- [Difference between Identifying and Non-Identifying Relationships](https://www.eandbsoftware.org/difference-between-identifying-and-non-identifying-relationships-mysql-workbench/)
- [The dotted/dashed line means, that SQL Server does not enforce referential integrity for this relationship](https://stackoverflow.com/questions/9347917/what-does-a-dashed-dotted-relationship-line-represent-in-sql-management-studio)
- [What's the difference between identifying and non-identifying relationships?](https://stackoverflow.com/questions/762937/whats-the-difference-between-identifying-and-non-identifying-relationships#762994)


## MySQL Workbench column flags

[SO](http://stackoverflow.com/questions/3663952/what-do-column-flags-mean-in-mysql-workbench) [Workbench](https://dev.mysql.com/doc/workbench/en/wb-table-editor-columns-tab.html)

    PK - Primary Key
    
    NN - Not Null
    
    BIN - Binary (stores data as binary strings. There is no character set so sorting and comparison is based on the numeric values of the bytes in the values.)
    
    UN - Unsigned (non-negative numbers only. so if the range is -500 to 500, instead its 0 - 1000, the range is the same but it starts at 0)
    
    UQ - Create/remove Unique Key
    
    ZF - Zero-Filled (if the length is 5 like INT(5) then every field is filled with 0’s to the 5th digit. 12 = 00012, 400 = 00400, etc. )
    
    AI - Auto Increment
    
    G - Generated column. i.e. value generated by a formula based on the other columns


# SQL Estándar
## Diferencia entre comillas simple y comillas doble en SQL

[Stack Overflow](https://stackoverflow.com/questions/1992314/what-is-the-difference-between-single-and-double-quotes-in-sql)

> A simple rule for us to remember what to use in which case
>    [S]ingle quotes are for [S]trings;
>    [D]ouble quotes are for [D]atabase identifiers;



## Verificar si un string tiene números 📰 

Para Luna, queríamos correr una rake task sobre las answers para encontrar una option_choice según el atributo content. El problema estaba en que varias answers eran solo dígitos (por error o por la pregunta Pain Scale).

Con estas consultas pude encontrar aquellas answers cuyo content contenía digitos.

Con esta no tuve resultados positivos:

    SELECT id, form_id, content
    FROM answers
    WHERE content LIKE '%[0-9]%';
    
     id | form_id | content 
    ----+---------+---------
    (0 rows)

Esta la encontré en [Stack Overflow](https://stackoverflow.com/questions/2558825/how-to-detect-if-a-string-contains-at-least-a-number).

Pero si hacía lo inverso:

    SELECT id, form_id, content
    FROM answers
    WHERE content NOT LIKE '%[0-9]%'
    LIMIT 10;
    
      id  | form_id |                                                            content                                                             
    ------+---------+--------------------------------------------------------------------------------------------------------------------------------
     1339 |       6 | I have no pain at the moment
     1340 |       6 | I can look after myself normally without causing extra pain
     1341 |       6 | I can lift heavy weights but it gives extra pain
     1342 |       6 | Pain prevents me from walking more than 1 mile
     1343 |       6 | I can only sit in my favourite chair as long as I like
     1344 |       6 | Pain prevents me from standing for more than 1 hour
     1345 |       6 | My sleep is occasionally disturbed by pain
       21 |       1 | The pain is very mild at the moment
       23 |       1 | Pain prevents me lifting heavy weights off the floor, but I can manage if they are conveniently placed, for example on a table
     1346 |       6 | My sex life is nearly normal but is very painful
    (10 rows)

obtenía resultados. Esto indicaba que la primera consulta no funciona como se espera por lo siguiente.


    SELECT id,content,REGEXP_MATCHES(content, '[[:digit:]]')
    FROM answers
    WHERE option_choice_id IS NULL;
    
      id  |                            content                             | regexp_matches 
    ------+----------------------------------------------------------------+----------------
     1342 | Pain prevents me from walking more than 1 mile                 | {1}
     1344 | Pain prevents me from standing for more than 1 hour            | {1}
       29 | My sleep is slightly disturbed (less than 1 hr sleepless)      | {1}
       24 | I can read as much as I want to with slight pain in my neck 2  | {2}
      276 | 2                                                              | {2}
      287 | 3                                                              | {3}
       44 | Pain prevents me from walking more than 1/2 mile               | {1}

Vista en [Stack Overflow](https://stackoverflow.com/questions/11341492/how-do-i-check-if-a-string-contains-a-number).

Vemos que la ID 276 es una de las que esperaríamos como respuesta de la 1era query y no fue así.

Sobre [REGEXP_MATCHES](https://www.postgresqltutorial.com/postgresql-string-functions/postgresql-regexp_matches/).


## Diferencia entre Time, Date y DateTime en base de datos que usa Rails

Ver en [Stack Overflow](https://stackoverflow.com/questions/3928275/in-ruby-on-rails-whats-the-difference-between-datetime-timestamp-time-and-da).

> The difference between different date/time formats in ActiveRecord has little to do with Rails and everything to do with whatever database you're using.

Ejemplo en MySQL:

- Date solo guarda fechas
- Time solo guarda horas
- DateTime guarda fecha y hora

**La Diferencia entre DateTime y Timestamp**
DateTime tiene el formato `YYYY-MM-DD HH:MM:SS`

> Valid ranges go from the year 1000 to the year 9999

Mientras que Timestamp es una representación del unix timestamp ([epoch time](https://en.wikipedia.org/wiki/Unix_time))

> Its valid range goes from 1970 to 2038

Además:

> TIMESTAMP used to **track changes of records, and update every time** when the record is changed. DATETIME used to **store specific and static value** which is not affected by any changes in records.

De este [artículo](https://stevenyue.com/blogs/date-time-datetime-in-ruby-and-rails/):

> Both **DateTime** and **Time** classes can be used to handle year, month, day, hour, min, sec attributes. But underneath, **Time** class stores integer numbers, which presents the seconds intervals since the Epoch. We also call it [**unix time**](http://en.wikipedia.org/wiki/Unix_time).


## Nested Conditions in Where clause

Cuál es la diferencia entre:

    WHERE (
      pat.latest_visit_date > to_iso8601(current_date - interval '90' day)
      OR
      pat.completed_visits_count = 0
    )

y

    WHERE pat.latest_visit_date > to_iso8601(current_date - interval '90' day)
      OR pat.completed_visits_count = 0

Más que diferencia es que según [Tech on the Net](https://www.techonthenet.com/sql_server/and_or.php):

> When combining these conditions, it is important to use parentheses so that the database knows what order to evaluate each condition. (Just like when you were learning the order of operations in Math class!)

Ahí la importancia de envolver en paréntesis las condiciones para producir los resultados esperados.

En [esta pregunta](https://stackoverflow.com/questions/12192137/where-clause-with-nested-multiple-conditions) en Stack Overflow muestran cómo usar condiciones anidadas.

    SELECT * 
    FROM testing
    WHERE (Location = 'Bhuj' AND Age < 20)
      OR (Location = 'Mumbai' AND Age > 25)

Y en este [SQL Fiddle](http://sqlfiddle.com/#!18/40c27/38) se puede probar.

