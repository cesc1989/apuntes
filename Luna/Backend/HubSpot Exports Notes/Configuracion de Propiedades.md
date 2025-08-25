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
> Como no tengo acceso a Omega tengo que pedir ayuda o emular la propiedad en mi cuenta dummy de HubSpot.
>
> La salida de la rake es el YML que habría que pegar en `config/hub_spot_contact_properties.yml`. Puedo copiarlo y pegarlo manualmente.


- Para actualizar Alpha

```bash
bundle exec rails hub_spot_contact_properties:upsert\[alpha\]
```