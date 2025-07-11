# Apuntes de Grimoire - Parte 1

## Añadiendo columnas nuevas en Edge

Ahora con lo de Logical Replication toca primero agregar las columnas en las bases de datos donde se replican las tablas de Edge. Normalmente, Grimoire.

Así se explica en [Syncing Data Between Services](https://www.notion.so/getluna/Syncing-Data-Between-Services-d8fe9a0902fc48598c567181a513de51#def49b59368d45bbad700b67f4291e94).


## Migraciones en Ecto

Así se hacen las [migraciones](https://hexdocs.pm/ecto_sql/Ecto.Migration.html#module-creating-your-first-migration) en [Ecto](https://stackoverflow.com/a/48495116/1407371) para agregar un campo:
```
mix ecto.gen.migration add_weather_table

mix ecto.gen.migration todos_add_author_column
```


Ejemplo:
```
mix ecto.gen.migration add_exercises_web_token_to_accounts
```

Correr todas las migraciones:
```
mix ecto.migrate
```