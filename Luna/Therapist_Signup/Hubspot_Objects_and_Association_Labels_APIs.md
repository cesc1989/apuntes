# Association Labels for Hubspot Contacts

Algunas notas sobre los endpoints para trabajar con la API de association labels de Hubspot.

Se usan los endpoints de la version 4 del API.

Todas estas peticiones llevan la siguiente cabecera enviando el private token para hacer peticiones a la API de Hubspot:

```
Authorization:Bearer [TOKEN]
```

## Listar Labels de Asociaciones

Antes de poder crear las asociaciones hay que crear las labels. Una vez creadas, se pueden acceder mediante el API con este endpoint:

```
GET https://api.hubapi.com/crm/v4/associations/:fromobjecttype/:toobjecttype/labels
```

Donde `:fromobjecttype` y `:toobjecttype` son `contact`.

Una respuesta luce similar a esto:
```json
{
  "results": [
    {
        "category": "USER_DEFINED",
        "typeId": 3,
        "label": "Referring Physician"
    },
    {
        "category": "USER_DEFINED",
        "typeId": 4,
        "label": "Referred Patient"
    },
    {
        "category": "HUBSPOT_DEFINED",
        "typeId": 449,
        "label": null
    },
    {
        "category": "USER_DEFINED",
        "typeId": 1,
        "label": "Reference"
    },
    {
        "category": "USER_DEFINED",
        "typeId": 2,
        "label": "Referencer"
    }
  ]
}
```

Aquí podemos ver como las _association labels_ funcionan en parejas. Para el caso de Professional References hay un _Referencer_ que se relaciona con un _Reference_. Por eso es que tenemos los `typeId` 1 y 2.

## Crear Professional References

Para la característica de Professional References se usaron los ==endpoints de la versión 4== de modo individual (la otra forma es en bache).

Para crear las asociaciones, en un ciclo, se va haciendo peticiones a este endpoint:
```
PUT https://api.hubapi.com/crm/v4/objects/:fromobjecttype/:fromobjectid/associations/:toobjecttype/:toobjectid

PAYLOAD
[
  {
    "associationCategory": "USER_DEFINED",
    "associationTypeId": 2
  }
]
```

Aquí `:fromobjecttype` y  `:toobjecttype` se reemplazan por `contact`.

Por otro lado, `:fromobjectid` y `:toobjecttype` son los IDs en Hubspot de cada referencia.

El cuerpo de la petición (payload) representa el tipo y la categoría (previamente creada) para la cual la petición hace la asociación. Estos valores son fijos.

### Respuestas

Cuando la petición es exitosa retorna una respuesta similar a esta:
```json
{
    "fromObjectTypeId": "0-1",
    "fromObjectId": 965951,
    "toObjectTypeId": "0-1",
    "toObjectId": 3019901,
    "labels": [
        "Referencer"
    ]
}
```

Si se envía una carga invalida, retorna algo como esto:
```json
{
    "status": "error",
    "message": "One or more associations are invalid",
    "correlationId": "59c85e80-fb8f-43eb-a926-46e342e16b5e",
    "context": {
        "associationSpec": [
            "USER_DEFINED=3"
        ],
        "INVALID_ASSOCIATION_TYPE_ID": [
            "USER_DEFINED=3 is not a valid association definition id"
        ]
    },
    "category": "VALIDATION_ERROR"
}
```

## Listar Associations

No tenía cabida dentro de la tarea listar las professional references pero para probar lo que hacía usaba este endpoint:
```
GET https://api.hubapi.com/crm/v4/objects/:fromobjectType/:objectid/associations/:toObjectType
```

Donde los parámetros de la URL tienen una dinámica similar a la petición PUT:

- `:fromobjectType` y `:toObjectType` son `contact`
- `:objectid` son el ID de contacto en Hubspot para el cual se quieren listar sus asociaciones por _label_

### Respuestas

Cuando hay asociaciones la respuesta JSON es similar a esta:
```json
{
  "results": [
    {
        "toObjectId": 12675602284,
        "associationTypes": [
            {
                "category": "HUBSPOT_DEFINED",
                "typeId": 449,
                "label": null
            },
            {
                "category": "USER_DEFINED",
                "typeId": 1,
                "label": "Reference"
            }
        ]
    },
    {
        "toObjectId": 12697710603,
        "associationTypes": [
            {
                "category": "HUBSPOT_DEFINED",
                "typeId": 449,
                "label": null
            },
            {
                "category": "USER_DEFINED",
                "typeId": 1,
                "label": "Reference"
            }
        ]
    }
  ]
}
```

# Custom Objects in Hubspot

La documentación dice que hay objetos estándar de Hubspot como Contacts, Companies, Deals, etc. También se pueden crear objetos custom para representar otro tipo de datos.

==Para trabajar con custom objects hay que usar la versión 3 de la API==.

Todas estas peticiones llevan la siguiente cabecera enviando el private token para hacer peticiones a la API de Hubspot:

```
Authorization:Bearer [TOKEN]
```

## Permisos para usar la API de Objects

Para trabajar con la API de Custom Objects, el token necesita alguno de estos scopes:

- crm.objects.custom.read
- crm.schemas.custom.read
- crm.objects.custom.write

## Listar custom objects de una cuenta

Para listar los custom objects de una cuenta hay que usar el [endpoint](https://developers.hubspot.com/docs/api/crm/crm-custom-objects#retrieve-existing-custom-objects):
```
GET https://api.hubapi.com/crm/v3/schemas
```

### Respuestas

La respuesta a la petición devuelve un array de objetos donde hay varias propiedades que a su vez son objetos o arrays:
```json
{
	"results": [
		{
		    "labels": {},
		    "requiredProperties": [],
		    "searchableProperties": [],
		    "primaryDisplayProperty": "state",
			"secondaryDisplayProperties": [],
			"description": "",
			"archived": false,
			"restorable": true,
			"metaType": "",
			"id": "30931773",
			"fullyQualifiedName": "p23451017_credentialing_2_0",
			"createdAt": "2024-06-04T18:17:43.627Z",
			"updatedAt": "2024-06-04T18:17:45.564Z",
			"createdByUserId": 28152039,
			"updatedByUserId": 28152039,
			"objectTypeId": "2-30931773",
			"properties": [],
			"associations": [],
			"name": "credentialing_2_0"
	    },
	    {
			"labels": {
			    "singular": "Therapist Address",
			    "plural": "Therapist Addresses"
			},
			"requiredProperties": [
			    "therapist_name"
			],
			"searchableProperties": [
			    "therapist_name",
			    "state",
			    "zip_code"
			],
			"primaryDisplayProperty": "therapist_name",
			"secondaryDisplayProperties": [
			    "state",
			    "zip_code"
			],
			"description": "Therapist Address object captures key information about a therapist's change of address and streamlines tracking their moves and credentialing requirements.",
			"archived": false,
			"restorable": true,
			"metaType": "PORTAL_SPECIFIC",
			"id": "30753436",
			"fullyQualifiedName": "p23451017_therapist_addresses",
			"createdAt": "2024-05-30T00:20:28.745Z",
			"updatedAt": "2024-05-30T00:20:30.248Z",
			"createdByUserId": 7007108,
			"updatedByUserId": 7007108,
			"objectTypeId": "2-30753436",
			"properties": [],
			"associations": [],
			"name": "therapist_addresses"
		}
	]
}
```

NOTA: los arrays `properties` y `associations` son enormes.

Cuando el token no tiene los scopes necesarios retorna una respuesta como esta:
```json
{
    "status": "error",
    "message": "This app hasn't been granted all required scopes to make this call. Read more about required scopes here: https://developers.hubspot.com/scopes.",
    "correlationId": "7b6c36b1-b505-4ddb-8715-2e5eb654416d",
    "errors": [
        {
            "message": "One or more of the following scopes are required.",
            "context": {
                "requiredScopes": [
                    "crm.objects.custom.read",
                    "crm.schemas.custom.read"
                ]
            }
        }
    ],
    "links": {
        "scopes": "https://developers.hubspot.com/scopes"
    },
    "category": "MISSING_SCOPES"
}
```


# 🌟 ¿Cómo acceder a los objetos custom asociados a un objeto Contact? 🌟

En la documentación para los objetos Contact veo este endpoint:
```
GET https://api.hubapi.com/crm/v3/objects/contacts/:contactid
```

Los docs dicen que se puede traer la lista de los objetos asociados enviando una lista de associations en los query params. Sin embargo, cuando traté con una petición así:
```
GET https://api.hubapi.com/crm/v3/objects/contacts/:contactid?associations=credentialing_2_0
```

retornó este error:
```json
{
  "status": "error",
  "message": "Unable to infer object type from: credentialing_2_0",
  "correlationId": "94d562a8-f8f9-4314-bea6-684bf3b7821a"
}
```

La forma correcta no es usando la propiedad `name` del objeto custom sino su propiedad `objectTypeId`.

Respuesta ejemplo de schema de objetos custom:
```json
{
	"results": [
		{
			"id": "30931773",
			"objectTypeId": "2-30931773",
			"properties": [],
			"associations": [],
			"name": "credentialing_2_0"
		}
	]
}
```

Y si en la petición usamos el `objectTypeId`:
```
GET https://api.hubapi.com/crm/v3/objects/contacts/:contactid?associations=2-30931773
```

La respuesta es:
```json
{
  "id": "47064462972",
  "properties": {
      "createdate": "2024-08-09T15:53:43.343Z",
      "email": "francisco.quintero+aftest@ideaware.co",
      "firstname": "Francisco",
      "hs_object_id": "47064462972",
      "lastmodifieddate": "2024-08-09T15:53:56.152Z",
      "lastname": "TEST"
  },
  "createdAt": "2024-08-09T15:53:43.343Z",
  "updatedAt": "2024-08-09T15:53:56.152Z",
  "archived": false,
  "associations": {
      "p23451017_credentialing_2_0": {
          "results": [
              {
                  "id": "14462501441",
                  "type": "contact_to_credentialing_2_0"
              }
          ]
      }
  }
}
```

Ahora la pregunta es, ==¿con qué endpoint obtengo detalles del ID que devuelve `results: []`?==

## Detalle de Objeto Custom

Con este endpoint:
```
GET https://api.hubapi.com/crm/v3/objects/:objectType/:objectId
```

Se puede traer el detalle de un Contact que corresponde a un Custom Object.

Así que tenemos el Contact con id `47064462972`. La petición que se hizo al endpoint `/contacts` devolvió una lista de asociaciones. En ese listado, cada objeto tiene un `id` que corresponde al objeto Contact para esa asociación (custom object).

Con eso en mente, tenemos que el id `14462501441` corresponde a un registro del objeto custom Credentialing 2.0. Para poder ver los detalles de este registro contact _custom_ tenemos que usar la misma API pero el endpoint cambia. En vez de ser la terminación _contacts_, ==la terminación pasa a ser el id del tipo de objeto custom==.

Así:
```
GET https://api.hubapi.com/crm/v3/objects/2-30931773/14462501441
```

responde con:
```json
{
  "id": "14462501441",
  "properties": {
      "hs_createdate": "2024-08-09T15:56:18.886Z",
      "hs_lastmodifieddate": "2024-08-09T15:56:18.886Z",
      "hs_object_id": "14462501441"
  },
  "createdAt": "2024-08-09T15:56:18.886Z",
  "updatedAt": "2024-08-09T15:56:18.886Z",
  "archived": false
}
```

Por defecto, devuelve una lista corta de propiedades. Para que devuelva otras hay que pasarle una lista a la petición en el query param:
```
GET https://api.hubapi.com/crm/v3/objects/:objectType/:objectId?properties=background_check_status
```

responde con:
```json
{
  "id": "14462501441",
  "properties": {
      "background_check_status": "Finding/Under Review",
      "hs_createdate": "2024-08-09T15:56:18.886Z",
      "hs_lastmodifieddate": "2024-08-13T21:10:17.567Z",
      "hs_object_id": "14462501441"
  },
  "createdAt": "2024-08-09T15:56:18.886Z",
  "updatedAt": "2024-08-13T21:10:17.567Z",
  "archived": false
}
```

# Actualizando Propiedades de un Objeto Custom

Usar este endpoint:
```
PATCH https://api.hubapi.com/crm/v3/objects/:objectType/:objectId
```

Ejemplo de petición:
```
PATCH https://api.hubapi.com/crm/v3/objects/2-30931773/14462501441

Payload

{
  "properties": {
    "bachelor_s_degree_school_name": "Hola Mundo Cachon"
  }
}
```

Y responde con:
```json
{
  "id": "14462501441",
  "properties": {
    "bachelor_s_degree_school_name": "Hola Mundo Cachon",
    "hs_created_by_user_id": "7007108",
    "hs_createdate": "2024-08-09T15:56:18.886Z",
    "hs_lastmodifieddate": "2024-08-16T04:46:56.541Z",
    "hs_object_id": "14462501441",
    "hs_object_source": "CRM_UI",
    "hs_object_source_id": "userId:7007108",
    "hs_object_source_label": "CRM_UI",
    "hs_object_source_user_id": "7007108",
    "hs_updated_by_user_id": "8338164"
  },
  "createdAt": "2024-08-09T15:56:18.886Z",
  "updatedAt": "2024-08-16T04:46:56.541Z",
  "archived": false
}
```

Devuelve la propiedad actualizada junto a otras extra.

# Enlaces y Documentación

## Referencer-Reference API docs

- Labels [https://knowledge.hubspot.com/object-settings/create-and-use-association-labels](https://knowledge.hubspot.com/object-settings/create-and-use-association-labels)
- Associations v4 [https://developers.hubspot.com/docs/api/crm/associations](https://developers.hubspot.com/docs/api/crm/associations)

## Custom Objects API docs

- [Knowledge Base article](https://knowledge.hubspot.com/object-settings/create-custom-objects#create-a-custom-object)
- CRM [Custom Objects](https://developers.hubspot.com/docs/api/crm/crm-custom-objects)
- [Define custom object associations](https://knowledge.hubspot.com/object-settings/create-custom-objects#create-a-custom-object)

## Contact Objects API docs

- Base API [docs](https://developers.hubspot.com/docs/api/crm/contacts)