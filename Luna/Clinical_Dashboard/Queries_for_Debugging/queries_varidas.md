# Query para Encontrar Registros Faltantes entre Dos Tablas o Bases de Datos

La query:

```sql
(SELECT 'patients' AS source, id, first_name, last_name FROM "luxe-production-db"."patients"
 EXCEPT
 SELECT 'patients' AS source, id, first_name, last_name FROM "luxe-production-db"."production")
UNION ALL
(SELECT 'production' AS source, id, first_name, last_name FROM "luxe-production-db"."production"
 EXCEPT
 SELECT 'production' AS source, id, first_name, last_name FROM "luxe-production-db"."patients")
```

Hay una disparidad de registros entre las dos bases de datos. **Con esa consulta pude encontrar los IDs que están en** `**production**` **que no están en** `**patients**`**.**

Visto en [Stack Overflow](https://stackoverflow.com/questions/2077807/sql-query-to-return-differences-between-two-tables/41150408#41150408).

# Query para mostrar Clinics con Patients en Alpha en Athena

```sql
SELECT count(pat.id),
 pat.clinic_id,
 cli.key
FROM "application-data"."patients" pat
join "application-data"."clinics" cli on cli.id = pat.clinic_id
where pat.clinic_id <> ''
group by pat.clinic_id, cli.key
order by cli.key;
```