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
```ruby
<HTTParty::Response:0xade0
    parsed_response=nil,
    @response=<Net::HTTPMethodNotAllowed 405 Method Not Allowed readbody=true>,
    @headers={
      "date" => ["Fri, 23 Jan 2026 16:45:51 GMT"],
      "transfer-encoding" => ["chunked"],
      "connection" => ["close"],
      "cf-ray" => ["9c28c38d7c47492b-BOG"],
      "cf-cache-status" => ["DYNAMIC"],
      "allow" => ["DELETE,GET,OPTIONS,PATCH"],
      "strict-transport-security" => ["max-age=31536000; includeSubDomains; preload"],
      "vary" => ["origin, Accept-Encoding"],
      "access-control-allow-credentials" => ["false"],
      "x-hubspot-correlation-id" => ["f5addc9e-2d34-4f60-8785-12cedbdd2747"],
      "set-cookie" => [
        "__cf_bm=kRH9GGB1cr1uqRv8yxp62z7S2BneiKDREG0fLxby6dw-1769186751-1.0.1.1-uCCWmyyomplFgJJawOgtI6aLUqGcWt1w.KbJ1gUtsfCgpPCpN9Kb9aUtS1V_wkzFxoZhVU5OoezUWra72DQ3N70iaCYmJLXQtIK_1kDHGWY; path=/; expires=Fri,
   23-Jan-26 17:15:51 GMT; domain=.hubapi.com; HttpOnly; Secure; SameSite=None"
      ],
      "report-to" => [
        "{\"endpoints\":[{\"url\":\"https://a.nel.cloudflare.com/report/v4?s=xBqDdKglnh5v411iTpH2gvUtI9PhqC98%2Fr4RZgcPyUvSiBrGDAIID8tWjrEiOPW0AQVTv9Pd4m0pH5YI3apTNiaOrtRyQcJDiWlaouFNDQWYUFHYJfZwJCz1XDS04aoa\"
  }],\"group\":\"cf-nel\",\"max_age\":604800}"
      ],
      "nel" => [
        "{\"success_fraction\":0.01,\"report_to\":\"cf-nel\",\"max_age\":604800}"
      ],
      "server" => ["cloudflare"]
    }
  >
```

Eso es porque la API de HubSpot no soporta actualizaciones con PUT sino con PATCH. Se ve en `"allow" => ["DELETE,GET,OPTIONS,PATCH"],`. 

Me tocó hacer la petición mediante HTTP.