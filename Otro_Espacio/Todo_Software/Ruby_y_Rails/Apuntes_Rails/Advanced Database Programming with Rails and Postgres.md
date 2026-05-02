# Advanced Database Programming with Rails and Postgres

De PG Analyze. Ver al final.

> Active Record is a little like a walled garden. It protects us as developers (and our users) from the harsh realities of what lies beyond those walls

> By understanding the SQL that Active Record is executing, we can open the gate in our walled garden to reach beyond what you may think is possible  
to accomplish in Rails, taking advantage of optimizations and flexibility that may be difficult to achieve otherwise.

El libro explica cómo usar subqueries en Active Record. Las subqueries que destacan son:

- Where
- From
- Select
- Having

## Where subquery

Ejemplo SQL
```sql
SELECT *  
FROM employees WHERE

employees.salary > ( SELECT avg(salary) FROM employees)
```

Esto puede hacerse en Ruby de la siguiente forma pero ejecutaría dos queries:
```ruby
Employee.where(‘salary > :avg’, avg: Employee.average(:salary))
```

Según el libro, esta forma sí lo hace en una sola query:
```ruby
Employee.where(‘salary > (:avg)’, avg: Employee.select(‘avg(salary)’))
```

Atención a que el placeholder `:avg` está entre paréntesis porque así lo pide el motor de la base de datos.

## Where Not Exists

Ejemplo SQL:
```ruby
SELECT employees.* FROM employees
LEFT OUTER JOIN vacations ON vacations.employee_id = employees.id WHERE vacations.id IS NULL
```

Así se podría traducir en Active Record:
```ruby
Employee.where(
  ‘NOT EXISTS (:vacations)’,
   vacations: Vacation.select(‘1’).where(‘employees.id = vacations.employee_id’)
)
```

## Select Subquery

Ejemplo del libro:
```sql
SELECT
  *,
  (SELECT avg(salary)
    FROM employees) avg_salary,
  salary - (
    SELECT avg(salary)
    FROM employees) above_avg
FROM employees
```

> Because the subquery is repeated, we can save ourselves a little bit of hassle by placing the subquery SQL into a variable that we‘ll embed into the outer query.

```ruby
avg_sql = Employee.select(‘avg(salary)’).to_sql

Employee.select(
  ‘*’,
  “(#{avg_sql}) avg_salary”,  
  “salary - (#{avg_sql}) avg_difference”
)
```

El mismo ejemplo pero usando Window Functions
```sql
SELECT
  *,
  avg(salary) OVER () AS avg_salary,
  salary - avg(salary) OVER () AS avg_salary
FROM
  employees
```

Que se escribe en Active Record:
```ruby
Employee.select(
  ‘*’,
  “avg(salary) OVER () avg_salary”,  
  “salary - avg(salary) OVER () avg_difference”
)
```

## The From subquery

_Pendiente_

# Views y Materialized Views

## Views

> A view allows us to query against the result of another query, providing a powerful way of abstracting away a complex query full of joins, conditions, groupings, and any other clause that can be added to an SQL query.

> **A regular view still performs the underlying query which defined it**. It will only be as efficient as its underlying query is.

## Materialized Views

> (...) they save the result of the original query to a cached/temporary table. **When you query a materialized view, you aren‘t querying the source data, rather the cached result**.


# Ebook Completo

  ![[Advanced.Database.Programming.with.Rails.pdf]]