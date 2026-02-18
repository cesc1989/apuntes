# NEX-3370 Unifica Propiedades en `HubspotContactService`

Etiquetas: #luna_help_desk 

**Reporte**
Las propiedades `credentialing_form_files` y/o `credentialing_form_url` no están disponibles para un Contacto de 2018.

## Contexto

Estas propiedades se actualizan desde diferentes servicios/workers porque en su momento tenían lógica diferente.

- `credentialing_form_files` se actualiza en `HubspotFormFilesService`
- `credentialing_form_url` se actualiza en `HubspotFormUrlService`

Además, la propiedad `attestation_form_url` se actualiza principalmente en los objetos Credentialing aunque también está disponible para los Contacts.

## Soluciones

### Mover todos las actualizaciones a `HubspotContactService`

Hay que mover lo que pasa en `HubspotFormFilesService` y en `HubspotFormUrlService` al servicio principal ya que en este se da el update inicial del contact.

Así también podemos salir de dos clases de servicios y de workers. Archivos a borrar:
- `app/services/credentialing/hubspot_form_files_service.rb`
- `app/services/credentialing/hubspot_form_url_service.rb`
- `app/workers/credentialing/hubspot_form_files_worker.rb`
- `app/workers/credentialing/hubspot_form_url_worker.rb`



### Generar prod-op para Contactos desactualizados

Contactos afectados:

| HS ID        | Email                           | Fixed? |
| ------------ | ------------------------------- | ------ |
| 73314451     | khartsoughdpt@gmail.com         |        |
| 5185156      | zaltug13@gmail.com              |        |
| 397151       | allegropt@aol.com               |        |
| 112688912257 | kfinucan@outlook.com            |        |
| 113216005642 | travis@reachphysicaltherapy.com |        |
| 120203084058 | mottola86@hotmail.com           |        |
| 160147419431 | sandra.stuckey@earthlink.net    |        |