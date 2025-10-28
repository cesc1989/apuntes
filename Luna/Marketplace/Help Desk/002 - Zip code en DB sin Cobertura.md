# 002 - Zip code en DB sin Cobertura

Etiquetas: #luna_help_desk 

Caso EDG-2905

Reporte: no se puede hacer un sign up para el Credentialing Application porque el zip code **20175** dice que no está en cobertura (respuesta de Marketplace) aunque sí es uno válido para Luna.

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
> Se necesita configurar un API key Google con Maps activado.

Con la API de Google correcta se puede hacer la petición y la respuesta sería:
```json
{
  "minimum_distance_meters": null,
  "resolved": {
    "location": {
      "address": {
        "city": "Leesburg",
        "formatted_address": "Leesburg, VA 20175, USA",
        "state": "VA",
        "street": null,
        "street_2": null,
        "zip_code": "20175"
      },
      "coordinates": {
        "latitude": 39.0910602,
        "longitude": -77.5536267
      }
    },
    "time_zone": "America/New_York"
  },
  "status": "unserviceable",
  "zip_code": "20175"
}
```

### El Problema: Geofence

El problema está en los límites del zip code. Al parecer la zona de esté no está cubierta en `app/marketplace/services/geofence.py` y por eso retorna que no está en zona de cobertura.

Así está la sección, que según Claude, produce el error:
```python
RegionIDs.POTOMAC: tuple([((38.6674333295, 39.2660114363), (-77.3233117612, -76.361898))]),
```

### La Solución: Ampliar el bounding box

Esta es la solución que ofrece Claudio:
```diff
- RegionIDs.POTOMAC: tuple([((38.6674333295, 39.2660114363), (-77.3233117612, -76.361898))]),
+ # Extended westward to -77.8783 to include Leesburg, VA (zip 20175)
+ RegionIDs.POTOMAC: tuple([((38.6674333295, 39.2660114363), (-77.8783, -76.361898))]),
```

Cuando vuelvo a probar la petición cambia la respuesta de `status`:
```json
{
  "minimum_distance_meters": null,
  "resolved": {
    "location": {
      "address": {
        "city": "Leesburg",
        "formatted_address": "Leesburg, VA 20175, USA",
        "state": "VA",
        "street": null,
        "street_2": null,
        "zip_code": "20175"
      },
      "coordinates": {
        "latitude": 39.0910602,
        "longitude": -77.5536267
      }
    },
    "time_zone": "America/New_York"
  },
  "status": "serviceable",
  "zip_code": "20175"
}
```

