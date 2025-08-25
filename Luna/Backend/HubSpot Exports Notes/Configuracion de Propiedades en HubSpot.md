# Configuración de Propiedades desde backend

Veo que hay dos archivos donde se listan las propiedades:

- `config/hub_spot_contact_properties.yml`
- `app/services/hub_spot_contact_properties.rb`

En los cambios para completar el export de Medicare Threshold Exceeded Claude hizo cambios al archivo YML. Eso está mal. Ese archivo no se actualiza manualmente sino mediante una rake. La rake es `lib/tasks/hub_spot_contact_properties.rake`.

> [!Note]
> Runbook - [Ver](https://www.notion.so/getluna/Syncing-New-HubSpot-Properties-1edd6a8a87b78035833fc3d1ad54d7fb)

Las instrucciones de uso de la rake se resumen en:

- Bajar acceso a HubSpot omega desde 1password
	- vault: "Engineering: Shared Logins Omega"
- Las nuevas propiedades se agregan manualmente a producción. Luego se bajan para actualizar el archivo YML y alpha.
- Para actualizar local

```bash
rm -f config/hub_spot_contact_properties.yml

bundle exec rails hub_spot_contact_properties:download_properties\[production\] > config/hub_spot_contact_properties.yml
```

> [!Note]
> Para obtener el token de Omega usando backend tengo que leer de la tabla `hubspot_tokens`.
>
> También puedo crear el campo en la cuenta dummy de HubSpot y usar el access token de esa cuenta. La salida de la rake es el YML que habría que pegar en `config/hub_spot_contact_properties.yml`. Puedo copiarlo y pegarlo manualmente.


- Para actualizar Alpha

```bash
bundle exec rails hub_spot_contact_properties:upsert\[alpha\]
```

# Tips para Crear Propiedades

Lo primordial es que los devs somos los que debemos crear las propiedades. Así controlamos mejor los valores que podremos exportar.

## El internal value en minúsculas

Procurar que el value o internal value esté en minúsculas. Esto para poderlo traducir a un símbolo en Ruby al hacer exports/imports.

## Preferir dropdown a radio

Funcionan igual pero dropdown es el preferido. En serio funcionan igual. En los docs se explican ambos con lo mismo.

[Ver](https://knowledge.hubspot.com/properties/property-field-types-in-hubspot#choosing-options)

Dropdown select: stores multiple options, where only one option can be selected as a value. In forms, they behave the same as radio select fields, but appear differently. This is an enumeration property. Learn more about technical limits of enumeration properties.