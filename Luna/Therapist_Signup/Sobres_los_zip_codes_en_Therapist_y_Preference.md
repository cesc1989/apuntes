# Sobres los zip codes en Therapist y Preference

Hay varios campos que representan zip_code en Therapist Signup. Están en los modelos Therapist y Preference.

Therapist
```
zip_code                                 :string
treating_postal_code                     :string
```

Preference
```
zip_code                                 :string
```

# Therapist zip_codes

El campo “zip_code” se usaba en el formulario de registro. Eso se cambió para usar el campo “treating_postal_code”.

En conclusión los campos quedan en este uso:

## Registro: `treating_postal_code`

![[reg.treazip.png]]

## Personal Information: `zip_code`

![[personal.info.zip.png]]

# Preference `zip_code`

Este campo no ha tenido cambios y se ha mantenido como desde el inicio.

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