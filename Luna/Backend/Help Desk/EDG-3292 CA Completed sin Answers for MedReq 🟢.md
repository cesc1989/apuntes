# EDG-3292 Therapists con CA Completo pero sin Answers en Medicare Requirement

Etiquetas: #luna_help_desk 

## Reporte

Ha habido casos recientes de Therapists que completan su CA pero luego no se genera el archivo PDF de la sección Medicare Requirements.

Cuando se inspeccionan los datos, resultan que no hay grabadas las respuestas. Es como si se saltaran esta sección e igual pudieran completar el form.

## Contexto

Esa sección es complicada de guardar.

Primero, las answers se crean luego del sign up. Pude comprobar que el Medicare Requirement existe con sus answer vacías.

Esto no es el problema.

Segundo, las pruebas y la revisión no muestran nada evidente. Cuando Fabricio inspeccionó LogRocket notó que para uno de los casos la petición de guardado devolvió un 422. Algo falló pero no pudimos ver los parámetros de la petición porque se usa LogRage.

## Queries de Grafana

Agregué una línea de log para poder revisar los parámetros. Estas son las queries de Grafana para encontrar todo lo relevante.

Con esta query puedo ver las líneas relevantes y así identificar el request ID.
```sql
{app="edge", environment="omega"}
  |= "2b39c99a-5c8d-4579-9a21-4e1864127ac4"
  |= "medicare_requirements"
```

Esta es la query para encontrar los parametros relevantes:
```ruby
{app="edge", environment="omega"}
  |= "[CREDENTIALING_DEBUG]"
```

# Soluciones

## Frontend

Esto parece ser un problema de frontend. Encontré una petición PUT cuya respuesta fue 422. Los parámetros de esa petición iban sin los esperados IDs de cada answer. Por eso no se actualizó nada y se devolvió el 422.