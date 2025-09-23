# 007 - Campo Preference#state rompe el form por estar en forma Abreviada

La lista de selección espera el nombre largo del estado. Al cargar el campo se rompe el form en la sección Preferences y no se pueden hacer más actualizaciones.

## Query para ver cuántos campos tienen la abreviatura

```sql
SELECT DISTINCT state, COUNT(*) as count
FROM preferences
WHERE state IS NOT NULL
  AND LENGTH(state) = 2
  AND state ~ '^[A-Z]{2}$'
GROUP BY state
ORDER BY state;
```

Resultados:

|state|count|
|-----|-----|
|AZ|69|
|CA|245|
|CO|79|
|DC|8|
|FL|66|
|GA|66|
|IL|36|
|IN|1|
|MA|17|
|MD|45|
|MI|29|
|MN|2|
|NC|42|
|NJ|1|
|NV|23|
|NY|31|
|OH|1|
|OK|7|
|OR|20|
|PA|28|
|TN|1|
|TX|107|
|UT|1|
|VA|22|
|WA|58|

Esto me parece que estaba así por alguna razón legacy que no recuerdo.

# Soluciones

## Backend - Webhook

Si se recibe en forma corta desde el webhook, convertir a la forma larga antes de guardar.

## Backend - Serializer

Convertir a la forma larga antes de enviar a frontend.