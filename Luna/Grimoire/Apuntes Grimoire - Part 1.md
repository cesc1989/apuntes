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

## Ecto naive_datetime vs utc_datetime_usec vs custom

`:naive_datetime`
```
grimoire_dev=# \d therapist_direct_access_entries
                   Table "public.therapist_direct_access_entries"
      Column       |              Type              | Collation | Nullable | Default
-------------------+--------------------------------+-----------+----------+---------
 id                | uuid                           |           | not null |
 therapist_id      | uuid                           |           |          |
 state_id          | uuid                           |           |          |
 permitted         | boolean                        |           |          |
 hubspot_id        | bigint                         |           |          |
 effective_from    | timestamp(0) without time zone |           |          |
 association_label | integer                        |           |          |
 deleted_at        | timestamp(0) without time zone |           |          |
 created_at        | timestamp(0) without time zone |           |          |
 updated_at        | timestamp(0) without time zone |           |          |
Indexes:
    "therapist_direct_access_entries_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "therapist_direct_access_entries_state_id_fkey" FOREIGN KEY (state_id) REFERENCES states(id)
    "therapist_direct_access_entries_therapist_id_fkey" FOREIGN KEY (therapist_id) REFERENCES therapists(id)
Publications:
    "app_publication"
```

`utc_datetime_usec`:
```
grimoire_dev=# \d therapist_direct_access_entries
                  Table "public.therapist_direct_access_entries"
      Column       |            Type             | Collation | Nullable | Default
-------------------+-----------------------------+-----------+----------+---------
 id                | uuid                        |           | not null |
 therapist_id      | uuid                        |           |          |
 state_id          | uuid                        |           |          |
 permitted         | boolean                     |           |          |
 hubspot_id        | bigint                      |           |          |
 effective_from    | timestamp without time zone |           |          |
 association_label | integer                     |           |          |
 deleted_at        | timestamp without time zone |           |          |
 created_at        | timestamp without time zone |           |          |
 updated_at        | timestamp without time zone |           |          |
Indexes:
    "therapist_direct_access_entries_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "therapist_direct_access_entries_state_id_fkey" FOREIGN KEY (state_id) REFERENCES states(id)
    "therapist_direct_access_entries_therapist_id_fkey" FOREIGN KEY (therapist_id) REFERENCES therapists(id)
Publications:
    "app_publication"
```

Custom `:"timestamp(6) without time zone"`
```
grimoire_dev=# \d therapist_direct_access_entries
                   Table "public.therapist_direct_access_entries"
      Column       |              Type              | Collation | Nullable | Default
-------------------+--------------------------------+-----------+----------+---------
 id                | uuid                           |           | not null |
 therapist_id      | uuid                           |           |          |
 state_id          | uuid                           |           |          |
 permitted         | boolean                        |           |          |
 hubspot_id        | bigint                         |           |          |
 effective_from    | timestamp(6) without time zone |           |          |
 association_label | integer                        |           |          |
 deleted_at        | timestamp(6) without time zone |           |          |
 created_at        | timestamp(6) without time zone |           |          |
 updated_at        | timestamp(6) without time zone |           |          |
Indexes:
    "therapist_direct_access_entries_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "therapist_direct_access_entries_state_id_fkey" FOREIGN KEY (state_id) REFERENCES states(id)
    "therapist_direct_access_entries_therapist_id_fkey" FOREIGN KEY (therapist_id) REFERENCES therapists(id)
Publications:
    "app_publication"
```