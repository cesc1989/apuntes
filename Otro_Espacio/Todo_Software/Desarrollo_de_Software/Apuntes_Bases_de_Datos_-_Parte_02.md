# Actualizar una columna con diferentes valores para varios registros

Para actualizar el campo `exercises_web_token` de Accounts mediante SQL y que cada registro tenga su propio token diferente lo puedo hacer así:

```sql
UPDATE accounts
SET exercises_web_token = substr(md5(random()::text || clock_timestamp()::text), 1, 16)
where id in (
  select acc.id
  from accounts acc
  inner join patients pat on pat.account_id = acc.id
  where acc.exercises_web_token is null
);
```

Nótese que para poder efectuar sobre solo los Accounts que son de Patient tuve que hacer el subquery en la clausula IN para poder hacer el INNER JOIN.

Una alternativa a:
```sql
substr(md5(random()::text || clock_timestamp()::text), 1, 16)
```

Es generar un UUID, así:
```
uuid_generate_v4();
# f47ac10b-58cc-4372-a567-0e02b2c3d479
```

Me lo dijo [chatgpt](https://chat.openai.com/share/461bc1de-a0f0-4815-898f-7d92fe35e40c)

# Data aggregation y funciones de agregación

Del artículo -> https://learnsql.com/blog/aggregate-functions/

> Data aggregation is the **process of taking several rows of data and condensing them into a single result or summary**. When dealing with large datasets, this is invaluable because ==it allows you to extract relevant insights without having to scrutinize each individual data point==.

> (...) functions that **perform calculations on groups of variables and return a single result**. (...) aggregate functions work on groups of rows of data. This allows you ==to efficiently compute statistics or generate summary information from a dataset==.

Las funciones más comunes son:

- SUM()
- COUNT()
- AVG()
- MIN()
- MAX()

# Saber el peso en disco de una base de datos PostgreSQL

Hay dos formas.

## Con una query:

```sql
SELECT pg_size_pretty(pg_database_size('db_name'));
```


## Con comandos de psql

```
\l+
```

Ejemplo:
```
luna_api_development_7  | francisco | UTF8     | 10 GB   | pg_default |
luna_api_development_8  | francisco | UTF8     | 11 GB   | pg_default |
```

Visto en [Stack Overflow](https://stackoverflow.com/questions/14346371/postgresql-find-total-disk-space-used-by-a-database).

# Usar `count` sin condicional where

Etiquetas: #luna_help_desk 

Esta query que le pedí a Claudio:
```sql
-- Count forms with completed_at in Backend
SELECT
    COUNT(*) as total_forms,
    COUNT(completed_at) as forms_with_completed_at,
    COUNT(*) - COUNT(completed_at) as forms_without_completed_at,
    ROUND(100.0 * COUNT(completed_at) / COUNT(*), 2) as percent_with_completed_at
FROM forms
WHERE progress_type = 'ongoing';
```

me llamó la atención esta línea:
```sql
COUNT(completed_at) as forms_with_completed_at,
```

¿Por qué no necesitaba de un `where` para contar solo los que tienen `completed_at` no nulo? Pues resulta que así funciona el `count` cuando le pasas el nombre de un campo. Según los [docs](https://www.postgresql.org/docs/current/functions-aggregate.html):
> `count` ( `"any"` ) → `bigint`
>
> Computes the number of input rows in which the input value is not null.

Me hizo recordar que Rafael Rodriguez (Totico) me explicó que es mejor contar con `count(id)` que con `count(*)` y es por esto.

# Columna en order by con expresión debe estar en select list

Me dio este error en una página de Luxe:
```
PG::InvalidColumnReference: ERROR:  for SELECT DISTINCT, ORDER BY expressions must appear in select list (ActionView::Template::Error)
LINE 1: ...appointments"."patient_practice_id" = $2 ORDER BY (SELECT MA...
```

Por esta query:
```ruby
class Episode
	def self.ordered_by_latest_scheduled_date
    order(
      Arel.sql(
        "(SELECT MAX(appointments.scheduled_date)
          FROM appointments
          WHERE appointments.episode_id = episodes.id) DESC NULLS LAST"
      )
    )
  end
end
```

en combinación con esta otra:
```ruby
class Patient
  has_many :patient_episodes,
           -> { distinct },
           class_name: "Episode",
           through: :patient_appointments,
           source: :episode
end
```

Causa el error ya que en `patient_episodes` hay uso de `distinct`. En ese caso el order que se ve en `ordered_by_latest_scheduled_date` tiene una expresión la cual se vuelve subquery y por lo tanto necesita especificar el campo en la lista en `select`.

Este sería el sql que produce Rails al combinar `patient_episodes.ordered_by_latest_scheduled_date` en la vista de Partners:
```sql
SELECT DISTINCT episodes.*
FROM episodes
INNER JOIN appointments ON appointments.episode_id = episodes.id
WHERE appointments.patient_practice_id = ?
ORDER BY (SELECT MAX(appointments.scheduled_date) -- <-- This ORDER BY expression
					FROM appointments
					WHERE appointments.episode_id = episodes.id) DESC NULLS LAST
```

Y por eso obtenemos el error. Lo puedo probar con estas queries sencillas.

Esta causa el error:
```sql
SELECT DISTINCT episodes.id, episodes.patient_id
FROM episodes
INNER JOIN appointments ON appointments.episode_id = episodes.id
ORDER BY appointments.scheduled_date DESC;
```

El error:
```
SQL Error [42P10]: ERROR: for SELECT DISTINCT, ORDER BY expressions must appear in select list

Position: 299
```

Esta es la versión corregida:
```sql
SELECT DISTINCT episodes.id, episodes.patient_id, appointments.scheduled_date
FROM episodes
INNER JOIN appointments ON appointments.episode_id = episodes.id
ORDER BY appointments.scheduled_date DESC;
```

# Instalar extensión pg_repack

Hay que descargar de [aquí](https://pgxn.org/dist/pg_repack/) como indica el [repo](https://github.com/reorg/pg_repack?tab=readme-ov-file).

Luego se siguen los comandos que dicen [los docs](https://reorg.github.io/pg_repack/):
```
cd pg_repack
make
sudo make install
```

Y debería quedar.

Así se activa en la consola de postgresql:
```sql
CREATE EXTENSION IF NOT EXISTS pg_repack;
```