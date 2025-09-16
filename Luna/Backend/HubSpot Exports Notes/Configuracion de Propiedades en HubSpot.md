# Configuración de Propiedades desde backend

Etiquetas: #luna_help_desk 

Veo que hay dos archivos donde se listan las propiedades:

- `config/hub_spot_contact_properties.yml`
- `app/services/hub_spot_contact_properties.rb`

En los cambios para completar el export de Medicare Threshold Exceeded Claude hizo cambios al archivo YML. Eso está mal. ==Ese archivo no se actualiza manualmente sino mediante una rake==.

La rake es `lib/tasks/hub_spot_contact_properties.rake`.

> [!Note]
> Runbook - [Ver](https://www.notion.so/getluna/Syncing-New-HubSpot-Properties-1edd6a8a87b78035833fc3d1ad54d7fb)

Las instrucciones de uso de la rake se resumen en:

- Actualiza `app/services/hub_spot_contact_properties.rb` con la nueva propiedad.
	- Ten en cuenta que están en orden alfabético.
- Las nuevas propiedades se crean manualmente en HubSpot en producción. Luego se bajan para actualizar el archivo `config/hub_spot_contact_properties.yml` y alpha.

> [!Tip]
> Para obtener el token de Omega usando `backend` tengo que leer de la tabla `hubspot_tokens`.

- Para actualizar local

```bash
rm -f config/hub_spot_contact_properties.yml

bundle exec rails hub_spot_contact_properties:download_properties\[production\] > config/hub_spot_contact_properties.yml
```

> [!Tip]
> También puedo crear el campo en la cuenta dummy de HubSpot y usar el access token de esa cuenta. La salida de la rake es el YML que habría que pegar en `config/hub_spot_contact_properties.yml`. Puedo copiarlo y pegarlo manualmente.
>
> Teniendo en cuenta el orden alfabético.


- Para actualizar Alpha

```bash
bundle exec rails hub_spot_contact_properties:upsert\[alpha\]
```

# Tips para Crear Propiedades

Lo primordial es que los devs somos los que debemos crear las propiedades. Así controlamos mejor los valores que podremos exportar.

## El internal value en minúsculas

Procurar que el value o internal value esté en minúsculas. Esto para poderlo traducir a un símbolo en Ruby al hacer exports/imports.

## 🌟 Preferir Dropdown select a Radio select 🌟

Funcionan igual pero dropdown es el preferido. En serio funcionan igual. En los docs se explican ambos con lo mismo.

[Ver](https://knowledge.hubspot.com/properties/property-field-types-in-hubspot#choosing-options)

> **Dropdown select**: stores multiple options, where only one option can be selected as a value. In forms, they behave the same as radio select fields, but appear differently. This is an enumeration property.

> **Radio select**: stores multiple options, where only one option can be selected as a value. *When editing on a record, they appear and behave the same as dropdown select fields*. In forms, they behave the same as dropdown select fields, but appear differently. This is an enumeration property.

## 🟡 Usar Single checkbox con cautela🟡

Solo guarda On/Off (true/false). Se muestra como el Radio select en la UI pero no permite agregar más valores.

Hay varias propiedades que usan este tipo de campo. Ejemplos:

- Mobile Account Is Deleted?
- Authorization Required?
- First Visit Cancelled?

Estas tienen la combinación:

- Label: Yes; Internal Name: `true`
- Label: No: Internal Name: `false`

Y así es más sencillo mandarles un valor booleano desde el código.