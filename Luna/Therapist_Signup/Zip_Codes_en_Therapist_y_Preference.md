# Sobres los Zip Codes en Therapist y Preference

Hay varios campos que representan Zip Code en Therapist Signup. Están en los modelos Therapist y Preference.

Therapist
```
zip_code                                 :string
treating_postal_code                     :string
```

Preference
```
zip_code                                 :string
```

# Campos Zip Code en Therapist

El campo `zip_code` se usaba en el formulario de registro. Eso se cambió para usar el campo `treating_postal_code`.

En conclusión los campos quedan en este uso:

## Registro: `treating_postal_code`

![[reg.treazip.png]]

## Personal Information: `zip_code`

![[personal.info.zip.png]]

# Preference `zip_code`

Este campo no ha tenido cambios y se ha mantenido como desde el inicio. Lo distinto de este campo es que el formulario siempre estuvo bloqueado. No se permitía a los usuarios editarlo. Esto implicó varios cambios en lógica desde del frontend.

Según el checkbox de "use home address as treatment address" se sobreescribía todo lo de esta sección y entonces el `Preference#zip_code` tomaba el valor de `Therapist#postal_or_zip_code`.

## Actualización, 26 de Junio 2025

Con la llegada del Attestation Form y todo el trabajo relacionado lo anterior en Preference cambia. Ahora el campo `zip_code` se puede editar así que hay que procurar usar el campo de este modelo y enviar este valor a HubSpot.

# Zip code en el Mini Form

En el mini form se pide `zip_code` en uno de los campos. ¿A qué `zip_code` se refiere?

En el blueprint puedo ver que se quiere usar el de Preference:
```ruby
field :zip_code do |therapist, _opts|
	therapist.preference&.zip_code
end
```

Pero en un form que abrí cargó vacío.

## Actualización, 9 de Abril 2024

Se cambió el código para que use el zip o treatment postal code del Therapist:

```ruby
field :zip_code do |therapist, _opts|
	therapist.postal_or_zip_code
end
```