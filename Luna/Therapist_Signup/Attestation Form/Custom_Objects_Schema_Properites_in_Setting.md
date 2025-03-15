# Attestation Form - Propiedades de Schemas Custom

Son tres propiedades que hay que guardar en general:

- name
- id 
- objectTypeId

Hay que guardarlas en el modelo `Setting` así que necesitamos una convención que soporte tanto los tres objetos custom de esta implementación como futuros.

A continuación, los **valores en Alpha** y las convenciones a usar por cada tipo de objeto.

## Para Credentialings

- name: `credentialings`
- id: `33642689`
- objectTypeId: `2-33642689`

**Convenciones**

- name: `credentialings_name`
- id: `credentialings_id`
- objectTypeId: `credentialings_objectTypeId`

## Para Licenses

- name: `licenses`
- id: `35178508`
- objectTypeId: `2-35178508`

**Convenciones**

- name: `licenses_name`
- id: `licenses_id`
- objectTypeId: `licenses_objectTypeId`

## Para Therapist Address

- name: `therapist_addresses`
- id: `33650059`
- objectTypeId: `2-33650059`

**Convenciones**

- name: `therapist_addresses_name`
- id: `therapist_addresses_id`
- objectTypeId: `therapist_addresses_objectTypeId`

