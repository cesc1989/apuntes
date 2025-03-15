# Migraciones con PostgreSQL constraints

Para empezar escribí esta migración que usa la función `exclusion_constraint`:
```ruby
add_exclusion_constraint :medicare_dollar_threshold_statuses, "tsrange(effective_from, effective_until) WITH &&, patient_id WITH =", using: :gist, name: "no_overlapping_medicare_dollar_threshold_statuses"
```

¿Qué significa todo eso?

La [sintaxis](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/PostgreSQL/SchemaStatements.html#method-i-add_exclusion_constraint) de la migración es:
```ruby
add_exclusion_constraint(table_name, expression, **options)
```

En la migración anterior el parámetro `expression` es este:
```ruby
"tsrange(effective_from, effective_until) WITH &&, patient_id WITH ="
```

## expression de `add_exclusion_constraint`

`tsrange` es un tipo de datos de PostgreSQL que representa un rango de timestamps. En este caso es entre los campos `effective_from` y `effective_until`.

Docs en las guías de Rails -> https://guides.rubyonrails.org/v7.1/active_record_postgresql.html#exclusion-constraints

Nótese que la sintaxis de la expresión es algo como:
```
expression1 WITH operator[, expression 2 WITH operator]
```

Ejemplos:
```ruby
tsrange(effective_from, effective_until) WITH &&
```

```ruby
patient_id WITH =
```

Docs de PostgreSQL [EXCLUDE](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-EXCLUDE).

Los operadores son:

- `&&`
- `=`

No sé que otras más habrá.

### Sobre los rangos

Los paréntesis no son por que sí. Son la forma de definir los límites de los rangos. Así lo [describen en los docs](https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-IO).

```
(lower-bound,upper-bound)
(lower-bound,upper-bound]
[lower-bound,upper-bound)
[lower-bound,upper-bound]
empty
```

> The parentheses or brackets indicate whether the lower and upper bounds are exclusive or inclusive

> In the text form of a range, an ==inclusive lower bound is represented by “`[`”== while an ==exclusive lower bound is represented by “`(`”==.
> 
> Likewise, an inclusive upper bound is represented by “`]`”, while an exclusive upper bound is represented by “`)`”

Por lo que en nuestra migración:
```ruby
"tsrange(effective_from, effective_until) WITH &&, patient_id WITH ="
```

Estamos diciendo que el rango sea exclusivo. Es decir que:

- El rango excluye el `upper_bound`.
- El valor de `effective_from` se incluye
- El valor de `effective_until` se excluye

Docs sobre los [límites de los rangos](https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-INCLUSIVITY).

### Qué hace `using: :gist`

ChatGPT dice:
> **GiST (Generalized Search Tree) index**, which is necessary for exclusion constraints involving range types like `tsrange`