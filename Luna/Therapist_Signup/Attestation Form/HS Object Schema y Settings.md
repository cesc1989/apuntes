# HS Object Schema y Settings

Cómo encontrar en `Setting` donde se guardan las propiedades de los schemas para los objetos Hubspot involucrados en el Attestation Form. Además de los schemas para los Association Labels.

Para poder trabajar con los Custom Objects de Hubspot, además de las Association Labels (para Personal References y Custom Objects), toca guardar propiedades del schema de Hubspot en la base de datos.

Normalmente son un `name` y un `id` que se guardan en el modelo `Setting`. Aquí trato de documentar todo eso para no tener que recordarlo tanto cada vez que toca hacer algo con alguno de estos custom objects.

## Listar los settings

Hay una rake task para listar los settings que estén en la base de datos y que están relacionados con los objetos Credentialings, Therapist Address y Licenses.

```
bundle exec rake hubspot:attestation:check_hs_custom_object_schemas
```

