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



# Enlaces y Documentación

Referencer-Reference API docs:

- Labels [https://knowledge.hubspot.com/object-settings/create-and-use-association-labels](https://knowledge.hubspot.com/object-settings/create-and-use-association-labels)
- Associations v4 [https://developers.hubspot.com/docs/api/crm/associations](https://developers.hubspot.com/docs/api/crm/associations)