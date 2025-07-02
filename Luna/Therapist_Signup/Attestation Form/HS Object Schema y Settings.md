# HS Object Schema y Settings

Cómo encontrar en `Setting` donde se guardan las propiedades de los schemas para los objetos Hubspot involucrados en el Attestation Form. Además de los schemas para los Association Labels.

Para poder trabajar con los Custom Objects de Hubspot, además de las Association Labels (para Personal References y Custom Objects), toca guardar propiedades del schema de Hubspot en la base de datos.

Normalmente son un `name` y un `id` que se guardan en el modelo `Setting`. Aquí trato de documentar todo eso para no tener que recordarlo tanto cada vez que toca hacer algo con alguno de estos custom objects.

## Listar los settings para esquemas de custom objects

Hay una rake task para listar los settings que estén en la base de datos y que están relacionados con los objetos Credentialings, Therapist Address y Licenses.

También ver [[Custom_Objects_Schema_Properites_in_Setting]]

```
bundle exec rake hubspot:attestation:check_hs_custom_object_schemas
```

Este comando devuelve esto:
```ruby
[
  {
    "credentialings_name" => "credentialings",
    "credentialings_id" => "33642689",
    "credentialings_objectTypeId" => "2-33642689",
    "credentialings_to_licenses_additional_type_id" => "147",
    "credentialings_to_licenses_treating_type_id" => "139",
    "credentialings_to_contact_relocation_type_id" => "60",
    "credentialings_to_contact_active_attested_type_id" => "52",
    "credentialings_to_contact_inactive_type_id" => "54",
    "credentialings_to_contact_active_type_id" => "62"
  },
  {
    "licenses_name" => "licenses",
    "licenses_id" => "35178508",
    "licenses_objectTypeId" => "2-35178508",
    "credentialings_to_licenses_additional_type_id" => "147",
    "credentialings_to_licenses_treating_type_id" => "139"
  },
  {
    "therapist_addresses_name" => "therapist_addresses",
    "therapist_addresses_id" => "33650059",
    "therapist_addresses_objectTypeId" => "2-33650059"
  }
]
```

Podemos ver los registros para los esquemas de los tres custom objects:
```ruby
"credentialings_name" => "credentialings",
"credentialings_id" => "33642689",
"credentialings_objectTypeId" => "2-33642689",

"licenses_name" => "licenses",
"licenses_id" => "35178508",
"licenses_objectTypeId" => "2-35178508",

"therapist_addresses_name" => "therapist_addresses",
"therapist_addresses_id" => "33650059",
"therapist_addresses_objectTypeId" => "2-33650059"
```

## Los settings para association labels

La rake anterior también nos devuelve los settings que tienen los ids para hacer las peticiones de Associations Labels.

De Credentialing a License:
```ruby
"credentialings_to_licenses_additional_type_id" => "147",
"credentialings_to_licenses_treating_type_id" => "139",
```

De Credentialing a Contact:
```ruby
"credentialings_to_contact_relocation_type_id" => "60",
"credentialings_to_contact_active_attested_type_id" => "52",
"credentialings_to_contact_inactive_type_id" => "54",
"credentialings_to_contact_active_type_id" => "62"
```

# Usar en código

Una cosa es listarlos con la rake y saber que existen y otra es usarlos en el código. Para eso hay que saber cual es el nombre lo cual no es muy intuitivo así que toca tenerlos a la mano.

## Crear/actualizar Custom Objects

Los endpoints para crear o actualizar custom objects necesitan el ID del Objeto padre.

Para obtener el object type ID para Licenses:
```ruby
Setting.find_by(key: "licenses_objectTypeId")&.value
```

Para obtener el object type ID para Credentialings:
```ruby
Setting.find_by(key: "credentialings_objectTypeId").value
```

Para obtener el object type ID para Therapist Addresses:
```ruby
Setting.find_by(key: "therapist_addresses_objectTypeId").value
```

## Para Association Labels

Los endpoints para asociar objetos de Hubspot necesitan el ID de la association label. Estos IDs corresponden según la dirección de la asociación.

Asociar Credentialing a License con label "Additional":
```ruby
Setting.find_by(key: "credentialings_to_licenses_additional_type_id").value
```

Asociar Credentialing a Contact con label "Active":
```ruby
Setting.find_by(key: "credentialings_to_contact_active_type_id").value
```

Asociar Credentialing a Contact con label "Active Attested":
```ruby
Setting.find_by(key: "credentialings_to_contact_active_attested_type_id").value
```

Asociar Credentialing a Contact con label "Relocation":
```ruby
Setting.find_by(key: "credentialings_to_contact_relocation_type_id").value
```


# Las rakes

Las rakes que guardan las propiedades para association labels están en el archivo `lib/tasks/015_save_hubspot_association_labels_to_settings.rake`.

Hay tres tareas:

```
hubspot:associations:save_labels_to_settings
hubspot:associations:save_credentialing_to_license_labels
hubspot:associations:save_credentialing_to_contact_labels
```

Por su parte, las rakes que guardan propiedades de los esquemas están en la rake `lib/tasks/019_save_custom_object_schema_properties.rake`.

Hay cuatro tareas:

```
hubspot:attestation:save_custom_object_schema_properties
hubspot:attestation:check_hs_custom_object_schemas
hubspot:attestation:check_custom_object_schemas_in_hubspot
hubspot:attestation:save_credentialing_schema_properties
```