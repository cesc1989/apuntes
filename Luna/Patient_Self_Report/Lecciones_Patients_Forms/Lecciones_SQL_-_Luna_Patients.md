# Lecciones SQL - Luna Patients

# Actualizar fecha con intervalo

Hay dos formas.

## Usando INTERVAL

```sql
UPDATE surgeries
SET date = date + INTERVAL '2000 year'
WHERE id = 2683;
```

Ejemplo

    ID: 32
    Fecha: 1111-01-11 00:00:00.000

Después del update:

    Fecha: 3111-01-11 00:00:00.000

## Usando MAKE INTERVAL

```sql
UPDATE surgeries
SET date = date + MAKE_INTERVAL(YEARS := 2000 - EXTRACT(YEAR FROM date)::INTEGER)
WHERE id = 2683;
```

Ejemplo

    ID: 538
    Fecha: 0198-12-01 00:00:00.000

Después del update:

    Fecha: 2000-12-01 00:00:00.000

