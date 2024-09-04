# Hubspot Search API - Notas

Esta es la API a usar para completar la funcionalidad de Unseen Patients for PBLs. Ya que los providers no tienen un hubspot_id ni están asociados a contacts de la forma en que sí lo están los physicians, toca es usar otras propiedades para buscar esos contactos.

Para poder encontrar a los contactos relacionados toca usar la API de búsqueda. Esta API tiene algunas diferencias clave con respecto a otras. Veamos.

## Importante

- Docs de la API https://developers.hubspot.com/docs/api/crm/search
- Hay que definir un valor para el parámetro `limit`
- Al usar el operador para filtros `IN`, la lista de valores deben ser en minúsculas
- En los filtros:
	- Para aplicar una lógica `AND` hay que definir varias condiciones en el operador `filters`
	- Para aplicar una lógica `OR` hay que definir varios `filters` en en un `filterGroup`

### Limitaciones

- It may take a few moments for newly created or updated CRM objects to appear in search results.
- You can include a maximum of five `filterGroups` with up to 10 `filters` in each group, with a maximum of 25 filters in total.
- The search endpoints are ==rate limited to four requests per second==.
- A **query can contain a maximum of 3,000 characters**. If the body of your request exceeds 3,000 characters, a 400 error will be returned.
- The ==search endpoints are limited to 10,000 total results for any given query==. Attempting to page beyond 10,000 will result in a 400 error.

## Usando la API

Ejemplo básico usando operador `EQ` (de _equal_). Aplicando lógica `AND` definiendo varias condiciones en un mismo filtro.

```json
// POST https://api.hubapi.com/crm/v3/objects/contacts/search

// PAYLOAD
{
  "limit": 100,
  "properties": [
    "firstname",
    "lastname",
    "email",
    "date_of_birth",
    "createdate",
    "hs_lead_status",
    "lead_source",
    "powered_by_luna_code"
  ],
  "filterGroups": [
    {
      "filters": [
        {
          "propertyName": "lead_source",
          "operator": "EQ",
          "value": "Powered by Luna"
        },
        {
          "propertyName": "powered_by_luna_code",
          "operator": "EQ",
          "value": "ACS"
        }
      ]
    }
  ]
}
```

Respuesta:
```json
{
  "total": 2,
  "results": [
    {
      "id": "5349",
      "properties": {
        "createdate": "2022-08-18T20:32:20.662Z",
        "date_of_birth": null,
        "email": "mrubinaz@comcast.net",
        "firstname": "Marshall",
        "hs_lead_status": "Patient - Qualified: Booked in Luxe",
        "hs_object_id": "5349",
        "lastmodifieddate": "2022-11-26T17:07:32.226Z",
        "lastname": "Rubin",
        "lead_source": "Powered by Luna",
        "powered_by_luna_code": "ACS"
      },
      "createdAt": "2022-08-18T20:32:20.662Z",
      "updatedAt": "2022-11-26T17:07:32.226Z",
      "archived": false
    }
  ]
}
```

### Operador IN

En el Admin Dashboard, se puede filtrar por varios Practices o Clinics. Esto significa que se puede buscar para varios proveedores diferentes. Para evitar tener que armar varios groups de `filterGroups` (además de que tiene una limitante), sale mejor usar el operador `IN`.

Nota que los valores están en minúsculas.

```json
{
  "limit": 100,
  "properties": [
    "firstname",
    "lastname",
    "email",
    "date_of_birth",
    "createdate",
    "hs_lead_status",
    "lead_source",
    "powered_by_luna_code"
  ],
  "filterGroups": [
    {
      "filters": [
        {
          "propertyName": "lead_source",
          "operator": "EQ",
          "value": "Powered by Luna"
        },
        {
          "propertyName": "powered_by_luna_code",
          "operator": "IN",
          "values": [
            "ucl",
            "acs"
          ]
        }
      ]
    }
  ]
}
```