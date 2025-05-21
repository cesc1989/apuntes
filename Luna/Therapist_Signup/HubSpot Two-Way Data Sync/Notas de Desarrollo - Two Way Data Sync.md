# Notas de Desarrollo - Two-Way data sync with HubSpot

# Campos con particularidades

El campo `registered_from` que actualiza la propiedad `license_issuing_state` del objeto License.

> [!Important]
> Esta propiedad también puede ser actualizada por el campo `state_name` del modelo License. Sin embargo, eso solo ocurre para el caso de los `License` en varios estados.
>
> Este documento trata sobre la propiedad para el caso del Physical Therapy License.

Cuando se completa el registro, se crea un objeto License en HubSpot. Una de las propiedades que se actualiza es `license_issuing_state` desde la clase payload `HubspotSignupPayload`.

```ruby
when :license
	state = UsaStatesHelper::ABBREVIATED[@therapist.registration_state]

	{
		"license_number": @therapist.physical_therapy_license_number,
		"license_expiration_date": @therapist.physical_therapy_license_expiration_date,
		"license_issuing_state": state
	}
end
```

Para el caso del Two-Way data sync, la propiedad debe ser actualizada por el campo `registered_from` del modelo `Therapist`.

---

Luego siguen las propiedades `name_change` y `aliases`. Estas no están soportadas porque el dato que se envía a HubSpot se obtiene de calcular un valor.

Para `name_change`:
```ruby
"name_change": @therapist.aliases.count.positive?
```

Para `aliases`:
```ruby
"aliases": @therapist.aliases.pluck(:name).join(", ")
```

Esta misma situación se da con la propiedad `licensed_in_other_states`:
```ruby
"licensed_in_other_states": @credentialing.licenses.count.positive? ? "Yes" : "No"
```

Si estas propiedades cambian en HubSpot, no se puede traducir ese cambio a algo directamente en la base de datos.

> [!Note]
> No tienen soporte para cambios sencillos. Sin embargo, se puede hacer que si el valor cambia a algo negativo, se borren los registros en los cuales se basó el cálculo inicial.
> 
> Ejemplo, si `aliases` se limpia, entonces borrar todos los `@therapist.aliases`.


# Probando Petición en Alpha

## Petición de campos Yes/No

Para este body:
```json
{"vid":"96251","properties":{"residency":{"value":"Yes"},"fellowship":{"value":"Yes"}}}
```

Tengo que armar el string así:
```
ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"96251","properties":{"residency":{"value":"Yes"},"fellowship":{"value":"Yes"}}}
```

Y luego pasarlo a SHA256 así:
```ruby
hash_string = 'ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"96251","properties":{"residency":{"value":"Yes"},"fellowship":{"value":"Yes"}}}'

Digest::SHA256.hexdigest(hash_string)
```

Esto genera el digest:
```
a02c3b73ba7b96f118ee7ece05a8b1c60d4c01d1929ef1bb44201563e8a22e0e
```

El cual se pega en el header `X-HubSpot-Signature` en Postman y se envía la petición.

