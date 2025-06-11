# Notas de Desarrollo - Two-Way data sync with HubSpot

# Campos con particularidades

## registered_from

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

## name_change

Luego siguen las propiedades `name_change` y `aliases`. Estas no están soportadas porque el dato que se envía a HubSpot se obtiene de calcular un valor.

Para `name_change`:
```ruby
"name_change": @therapist.aliases.count.positive?
```

## aliases

Para `aliases`:
```ruby
"aliases": @therapist.aliases.pluck(:name).join(", ")
```

## licensed_in_other_states

Esta misma situación se da con la propiedad `licensed_in_other_states`:
```ruby
"licensed_in_other_states": @credentialing.licenses.count.positive? ? "Yes" : "No"
```

Si estas propiedades cambian en HubSpot, no se puede traducir ese cambio a algo directamente en la base de datos.

> [!Note]
> No tienen soporte para cambios sencillos. Sin embargo, se puede hacer que si el valor cambia a algo negativo, se borren los registros en los cuales se basó el cálculo inicial.
> 
> Ejemplo, si `aliases` se limpia, entonces borrar todos los `@therapist.aliases`.

## start_date_therapist

Esta se actualiza con el campo `care_start_date` de Therapist. Sin embargo, luego también se puede actualizar por el campo `projected_start_date` de Preference.

Para cuando se actualice mediante el webhook hay que actualizar ambos campos en la bd.

## Campos de Address: Home y Treatment

En la sección Personal Information están los campos. Estos son *Home Address*.

| Título               | DB field               | HS Contact property           |
|----------------------|------------------------|-----------------------|
| Home Address         | `home_address`         | address               |
| Apartment            | `street_address_line_2`| street_address_line_2 |
| State                | `state`                | state                 |
| County               | `county_of_residence`  |                       |
| City                 | `city`                 | city                  |
| Zip Code             | `zip_code`             | zip                   |

En cambio, en la sección Preference, hay campos similares pero para *Treatment Address*.

| Título      | DB field               | HS Contact property             |
|-------------|------------------------|-------------------------|
| Street      | `street`               | treating_street_address |
| Apartment   | `street_address_line_2`|                         |
| State       | `state`                |                         |
| City        | `city`                 | treating_city           |
| Zip Code    | `zip_code`             | treating_postal_code    |


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

# Propiedades de Personal Reference y Professional Reference

Ambos grupos de propiedades son del mismo dato en la BD (PersonalReference) pero el término Professional Reference es el preferido por el Success Team.

## Los grupos de propiedades para Personal Reference

Estos están en el HS Contact.

```
personal_reference_id
personal_reference_name
personal_reference_email
personal_reference_phone
personal_reference_title

personal_reference_id_2
personal_reference_name_2
personal_reference_email_2
personal_reference_phone_2
personal_reference_title_2

personal_reference_id_3
personal_reference_name_3
personal_reference_email_3
personal_reference_phone_3
personal_reference_title_3
```

En este grupo las propiedades de la referencia #1 no tienen el sufijo en el nombre porque inicialmente las crearon así.

## Los grupos de propiedades para Professional Reference

Estos viven en el HS Credentialing object.

```
professional_reference_name_1
professional_reference_email_1
professional_reference_phone_1
professional_reference_title_1
professional_reference_id_1
professional_reference_affiliation_start_date_1
professional_reference_affiliation_end_date_1

professional_reference_name_2
professional_reference_email_2
professional_reference_phone_2
professional_reference_title_2
professional_reference_id_2
professional_reference_affiliation_start_date_2
professional_reference_affiliation_end_date_2

professional_reference_name_3
professional_reference_email_3
professional_reference_phone_3
professional_reference_title_3
professional_reference_id_3
professional_reference_affiliation_start_date_3
professional_reference_affiliation_end_date_3
```