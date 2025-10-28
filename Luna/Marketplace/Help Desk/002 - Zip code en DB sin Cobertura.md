# 002 - Zip code en DB sin Cobertura

Etiquetas: #luna_help_desk 

Caso EDG-2905

Reporte: no se puede hacer un sign up para el Credentialing Application porque el zip code 20175 dice que no está en cobertura (respuesta de Marketplace) aunque sí es uno válido para Luna.

## Contexto

Así pude comprobar que el zip code sí está en la base de datos:
```sql
SELECT *
FROM marketplace.treating_zip_code
WHERE value = '20175';
```

Claudio sugirió que el zip podría estar en una zona no soportada. Con esta query se puede revisar.
```sql
SELECT key, value, type
FROM marketplace.setting
WHERE key = 'supported_states';
```

Retorna:
```
AZ,CA,CO,CT,DC,DE,FL,GA,IL,IN,MA,MD,MI,MN,MO,NC,NV,NY,OH,OK,OR,PA,TN,TX,UT,VA,WA,WI
```

Dado que el zip code está en el estado de Virginia sí se espera que esté habilitado.

### El endpoint

Está en el archivo `app/marketplace/serviceability/blueprint.py`. Función `validate_zip_code_is_in_area`.

Para probar en local se puede hacer esta petición curl:
```bash
curl --request POST \
  --url http://127.0.0.1:5000/api/v1/internal/validate-zip-code-is-in-area \
  --header 'authorization: Token holatoken' \
  --header 'content-type: application/json' \
  --data '{
  "zip_code": 20175
}'
```

> [!Tip]
> Se necesita configurar un API key de Google Maps.

### El Problema: Geofence

El problema está en los límites del zip code. Al parecer la zona de esté no está cubierta en `app/marketplace/services/geofence.py` y por eso retorna que no está en zona de cobertura.