# Attestation Form - Guardar Propiedades de Schemas Custom

Son tres propiedades que hay que guardar en general:
- name
- id 
- objectTypeId

Hay que guardarlas en Setting así que necesitamos una convención que soporte tanto los tres objetos custom de esta implementación como futuros.

A continuación, los valores de prueba y las convenciones a usar por cada tipo de objeto.

## Para Credentialings

- name: `credentialings`
- id: `33642689`
- objectTypeId: `2-33642689`

**Convenciones**

- name: `credentialings_address_name`
- id: `credentialings_address_id`
- objectTypeId: `credentialings_object_type_id`

## Para Licenses

- name: `licenses`
- id: `35178508`
- objectTypeId: `2-35178508`

**Convenciones**

- name: `licenses_address_name`
- id: `licenses_address_id`
- objectTypeId: `licenses_object_type_id`

## Para Therapist Address

- name: `therapist_addresses`
- id: `33650059`
- objectTypeId: `2-33650059`

**Convenciones**

- name: `therapist_address_name`
- id: `therapist_address_id`
- objectTypeId: `therapist_object_type_id`