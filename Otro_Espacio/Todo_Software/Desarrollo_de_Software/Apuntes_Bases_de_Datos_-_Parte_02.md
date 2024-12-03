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