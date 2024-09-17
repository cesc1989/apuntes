# Unseen Patients for Physicians - Notes

Parece que cuando hice este trabajo no tome ningún tipo de apuntes en ningún lado (????) y ahora que debo hacer una actualización no tengo forma de recordar nada. Entonces aquí voy a describir todo el flujo para tener un punto de partida cada que toque trabajar en esta característica.

# Clases Involucradas

- UnseenPatientsController
	- recibe el parámetro `physician_hubspot_ids`
- UnseenPatients::UnseenPatientsInformation
	- Esta es la clase general para todo tipo de Provider
- UnseenPatients::AssociatedContacts
	- Para recolectar los `hubspot_ids` de los contactos asociados al (los) physicians
- UnseenPatients::ReferredPatientsInformation
	- Donde se hace la petición al API `crm/v3/objects/contacts/batch/read`
- UnseenPatients::Categorized
	- Esta clase se encarga de agrupar los datos para la gráfica de barras


# Al Detalle

> [!INFO] Todo esto funciona a partir de que cada physician se puede identificar mediante un `hubspot_id`.

## UnseenPatientsController

En el parámetro `physician_hubspot_ids` se envía uno (si es Dashboard de Physician) o varios (si es Admin Dashboard) `hubspot_ids` para los cuales buscar los contactos (patients) asociados mediante _Association Label_ los cuáles son los _Unseen Patients_.

Ejemplo:
```json
// POST /v1/unseen_patients

// Payload
{
  "data": {
    "physician_hubspot_ids": [
      19976815111,
      265101,
      9101
    ],
    "practices_codes":[],
    "clinics_codes": []
  }
}
```

## UnseenPatients::UnseenPatientsInformation

En esta clase pasan varias cosas:

1. Se determina que flujo seguir para buscar los contactos en Hubspot según el Provider
2. Cuando es Physician:
	1. Se usa la clase `UnseenPatients::AssociatedContacts` para encontrar los `hubspot_ids` de los contactos asociados
	2. Se pasan estos `hubspot_ids` a la clase `UnseenPatients::ReferredPatientsInformation` donde se hace la petición para traer los detalles de cada uno.
3. Con los datos de cada contacto:
	1. Se compila la lista para mostrar en la tabla
	2. Se calculan los valores para la gráfica de Barras

## UnseenPatients::AssociatedContacts

Esta clase hace una petición al endpoint:
```
crm/v4/objects/:fromobjectType/:objectid/associations/:toObjectType
```

Donde:

- `:fromobjectType` y `:toObjectType` son igual a `contact`
- `:objectid` es el `hubspot_id` del Physician para el cual pedir los contactos

La respuesta de esta petición se limpia para solo conservar los resultados cuya `label` de asociación sea `Referred Patient`.

Ejemplo, en esta respuesta:
```json
{
  "results": [
    {
      "toObjectId": 12675602284,
      "associationTypes": [
        {
          "category": "USER_DEFINED",
          "typeId": 1,
          "label": "Reference"
        },
        {
          "category": "HUBSPOT_DEFINED",
          "typeId": 449,
          "label": null
        }
      ]
    },
    {
      "toObjectId": 13947063307,
      "associationTypes": [
        {
          "category": "USER_DEFINED",
          "typeId": 4,
          "label": "Referred Patient"
        },
        {
          "category": "HUBSPOT_DEFINED",
          "typeId": 449,
          "label": null
        }
      ]
    }
  ]
}
```

El paso de limpiar es dejar solo los objetos que cumplan con la condición mencionada así que el resultado final sería:
```json
{
	"toObjectId": 13947063307,
	"associationTypes": [
		{
			"category": "USER_DEFINED",
			"typeId": 4,
			"label": "Referred Patient"
		},
		{
			"category": "HUBSPOT_DEFINED",
			"typeId": 449,
			"label": null
		}
	]
}
```

## UnseenPatients::ReferredPatientsInformation

Aquí se hace la última petición que es para obtener los detalles de cada contacto (patient) y poder mostrar los widgets en el dashboard.

Antes de hacer la petición, hay que filtrar los `hubspot_id` de cada contacto. Para hacerlo hay que usar la respuesta que arroja la clase `UnseenPatients::AssociatedContacts`. De este JSON:
```json
{
	"toObjectId": 13947063307,
	"associationTypes": [
		{
			"category": "USER_DEFINED",
			"typeId": 4,
			"label": "Referred Patient"
		},
		{
			"category": "HUBSPOT_DEFINED",
			"typeId": 449,
			"label": null
		}
	]
}
```

Se usa la propiedad `toObjectId`.

> [!WARNING]
> En este paso se divide la cantidad de parámetros en grupos de 100 porque ese es el límite permitido por la API de Hubspot.

Con los `hubspot_ids` agrupados en listas, se hace la petición al endpoint:
```json
// POST /crm/v3/objects/contacts/batch/read

// Payload
{
  "properties": [
    "firstname",
    "lastname",
    "email",
    "date_of_birth",
    "createdate",
    "hs_lead_status"
  ],
  "inputs": [
    {
      "id": "13947063307"
    },
    {
      "id": "13948203075"
    }
  ]
}
```

El resultado final es el que usa la clase `UnseenPatients::UnseenPatientsInformation` para generar la lista para la tabla y pasar a la clase `UnseenPatients::Categorized` para generar los datos para el gráfico de barras.

## UnseenPatients::Categorized

_Pendiente_