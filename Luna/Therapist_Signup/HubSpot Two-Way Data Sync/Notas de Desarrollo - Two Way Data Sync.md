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