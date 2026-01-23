# EDG-3303 Siempre haz sync de las Additional Licenses sea CA o AF

Etiquetas: #luna_help_desk 

## Reporte

Un therapist agregó licencias adicionales en el AF y no se crearon en HubSpot.

## Problema

Resulta que la creación solo estaba configurada desde el Credentialing Application. En el AF solo se hacía update.

## Solución

Siempre sincronizar. Hacer un create/update desde ambos forms para simplificar el proceso.

### Nota con respecto a `Hubspot::Connection.put_json`

Quería hacer esto para actualizar los license objects en HubSpot:
```ruby
::Hubspot::Connection.put_json(update_path, params: {}, body: update_body)
```

pero obtenía el error:
```
Net::HTTPMethodNotAllowed 405 Method Not Allowed
```

Eso es porque la API de HubSpot no soporta actualizaciones con PUT sino con PATCH.

Me tocó hacer la petición mediante HTTP.