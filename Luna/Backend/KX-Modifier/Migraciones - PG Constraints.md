# Migraciones con PostgreSQL constraints

Para empezar escribí esta migración que usa la función `exclusion_constraint`:
```ruby
add_exclusion_constraint :medicare_dollar_threshold_statuses, "tsrange(effective_from, effective_until) WITH &&, patient_id WITH =", using: :gist, name: "no_overlapping_medicare_dollar_threshold_statuses"
```

> [!Note]
> La restricción se pone para prevenir que exista más de un registro `medicare_dollar_threshold_statuses` para un mismo año para un mismo paciente.

¿Qué significa todo eso?

La [sintaxis](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/PostgreSQL/SchemaStatements.html#method-i-add_exclusion_constraint) de la migración es:
```ruby
add_exclusion_constraint(table_name, expression, **options)
```

En la migración anterior el parámetro `expression` es este:
```ruby
"tsrange(effective_from, effective_until) WITH &&, patient_id WITH ="
```

## expression en `add_exclusion_constraint`

`tsrange` es un tipo de datos de PostgreSQL que representa un rango de timestamps. En este caso es entre los campos `effective_from` y `effective_until`.

Docs en las guías de Rails -> https://guides.rubyonrails.org/v7.1/active_record_postgresql.html#exclusion-constraints

Nótese que la sintaxis de la expresión es algo como:
```
expression1 WITH operator[, expression 2 WITH operator]
```

En la query tenemos esto:
```ruby
tsrange(effective_from, effective_until) WITH &&
```

Donde `tsrange(effective_from, effective_until)` crea un rango de timestamps usando los dos valores. Y por su parte `WITH &&` asegura que no haya dos registros donde se solapen los rangos.

O sea que puede haber dos registros cuyo rango de timestamps sea:

- MDTS 1: 2024-01-01 00:00:00 - 2024-06-30 23:59:59
- MDTS 2: 2023-01-01 00:00:00 - 2023-06-30 23:59:59

Pero no puede haber dos registros donde se solape alguno de los límites de los rangos.

Por su parte:
```ruby
patient_id WITH =
```

Es para asegurar que la restricción aplique por paciente. Solo un MDTS puede existir por paciente en un rango de `effective_from` y `effective_until`.

Docs en PostgreSQL [EXCLUDE constraint al crear tabla](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-EXCLUDE).

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