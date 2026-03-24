# Apuntes Ciclo 14 - Integración con Bold

Todo lo que aprendí de esta integración.

## ENVs que hay que configurar

Dos:
```bash
ENLACITO_BOLD_API_KEY="identity_key"
ENLACITO_APP_HOST="https://enlacito.co"
```

## La URL del POST debe terminar sin barra

El endpoint debe ser como en lo verde de este diff:
```diff
-https://integrations.api.bold.co/online/link/v1/
+https://integrations.api.bold.co/online/link/v1
```

Si queda la barra al final dará una respuesta de que no se puede hacer la petición.

## La URL callback no puede ser HTTP ni localhost

Tenía esto en la propiedad callback_url:
```
localhost:3005/pro
```

Causaba que la petición fallara con un 403:
```json
{
  "message": "Forbidden"
}
```

Para solventar aquí tocó usar ngrok.

### Usando ngrok para https local

Ya instalado y configurado según indican los docs corrí:
```bash
ngrok http 3005
```

Luego tocó agregar el dominio generado en `config.hosts` en development.rb:
```ruby
config.hosts << "outbully-unmoldy-corrie.ngrok-free.dev"
```

Ya con eso cambié el valor de `APP_HOST`:
```bash
APP_HOST="https://outbully-unmoldy-corrie.ngrok-free.dev"
```

Una vez esto ya funcionaba hacer la petición a Bold.

## Redirección de Bold hacía Enlacito

Bold aporta unos parámetros en la URL del callback. Ejemplo:
```
https://outbully-unmoldy-corrie.ngrok-free.dev/pro?bold-order-id=enlacito_pro_1_1774239020&bold-tx-status=approved
```

Con esos en la vista se puede trabajar para determinar si el pago fue exitoso.