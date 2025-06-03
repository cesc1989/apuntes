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

## Query de Graphana para que no salgan logs molestos

Usarla antes de correr queries para limpiar bastante la salida.

```
{app="therapist-credentialing-backend"} != `path=/okcomputer/all.json` != `SELECT "schema_migrations"`
```

## Petición de campos Yes/No

Para este body:
```json
{"vid":"121819130957","properties":{"residency":{"value":"Yes"},"fellowship":{"value":"Yes"}}}
```

Tengo que armar el string así:
```
ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"96251","properties":{"residency":{"value":"Yes"},"fellowship":{"value":"Yes"}}}
```

Y luego pasarlo a SHA256 así:
```ruby
hash_string = 'ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"121819130957","properties":{"residency":{"value":"Yes"},"fellowship":{"value":"Yes"}}}'

Digest::SHA256.hexdigest(hash_string)
```

Esto genera el digest:
```
21ffe3fdb866fbd4c35e48ae2234079d373e597c791513cfa1bc145781154128
```

El cual se pega en el header `X-HubSpot-Signature` en Postman y se envía la petición.


## Petición de Treating state/city & zip code

A estos tocó ponerles un prefijo para diferenciarlos.

```json
{"vid":"121819130957","properties":{"state_treating":{"value":"CA"},"city_treating":{"value":"Los Angeles"},"zip_code":{"value":"101010"}}}
```

Generar el digest:
```ruby
hash_string = 'ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"121819130957","properties":{"state_treating":{"value":"CA"},"city_treating":{"value":"Los Angeles"},"zip_code":{"value":"101010"}}}'

Digest::SHA256.hexdigest(hash_string)
```

Resultado:
```
8e8e50866ca03bc8d9f4693cd0c1bb0466866eb37fee526cd0fd9a2fdfee7212
```

## Petición de Personal References

```json
{"vid":"101262458469","properties":{"personal_reference_id":{"value":628},"personal_reference_name":{"value":"Capy Bara 1"},"personal_reference_email":{"value":"capybara_1@gmail.com"},"personal_reference_phone":{"value":"(415) 222-3434"},"personal_reference_title":{"value":"Physical Therapist"}}}
```

Generar el digest:
```ruby
hash_string = 'ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"101262458469","properties":{"personal_reference_id":{"value":628},"personal_reference_name":{"value":"Capy Bara 1"},"personal_reference_email":{"value":"capybara_1@gmail.com"},"personal_reference_phone":{"value":"(415) 222-3434"},"personal_reference_title":{"value":"Physical Therapist"}}}'

Digest::SHA256.hexdigest(hash_string)
```

Resultado:
```
53a2d6def53e48689d075f65c7198ff902b34857e36bbdee5f83a0aad4ed30a0
```

### Probar cambiar el valor de un atributo

Al cambiar algo hay que volver a generar el digest. Probemos que el `personal_reference_name` cambie.

Generar el digest:
```ruby
hash_string = 'ed9a2369-5118-44a1-9a91-39e31f789c03POSThttps://therapist-signup.alpha.getluna.com/v2/external/therapist_hubspot_webhook{"vid":"101262458469","properties":{"personal_reference_id":{"value":628},"personal_reference_name":{"value":"Mugi Wara"},"personal_reference_email":{"value":"capybara_1@gmail.com"},"personal_reference_phone":{"value":"(415) 222-3434"},"personal_reference_title":{"value":"Physical Therapist"}}}'

Digest::SHA256.hexdigest(hash_string)
```

Resultado:
```
e58e526e6627a5e372c214e8566387d5684f2263f6ee59c32253602fb33b98fd
```