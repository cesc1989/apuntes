# 006 - Credentialing object no recibe sync desde CA

Varios reportes se han amontonado donde el Credentialing object no recibe datos desde el CA.

## Contexto

Para poder hacer updates el HS Credentialing object se necesita el ID de este objeto. Dicho ID se guarda en la columna `credentialing_active_attested_id`. Dado a que los custom objects son creados por workflows en HubSpot hay un retraso incierto entre cuando se crea el Credentialing object y se hace la petición para conseguir el ID.

Para gestionar dicho retraso, cuando se completa in signup se ejecutan varios workers. Así configuré el que hace el sync de los IDs:
```ruby
# Sync HS Custom Object IDs as soon as possible
#
# We don't know how long it takes Hubspot to create Custom Objects. It's usually fast.
# Let's give it 60 seconds before trying to pull Credentialing and Therapist Address IDs.
HubspotSyncCustomObjectIdsToTherapistWorker.perform_in(60.seconds, therapist.id)
```

Se ejecuta 60 segundos después de agendado para darle espacio al workflow en HubSpot para que se complete. Sin embargo, esto parece on funcionar del todo dado a que durante los multiples intentos de sincronizar durante el CA terminan en reportar el error en Sentry:
```
HubspotCustomObjects::HubspotCredentialingObjectSyncException
Missing definitive_hubspot_credentialing_id for Therapist [CONTACT_ID]
```

Eso quiere decir que el sync no logró obtener el ID ni guardarlo en la BD.

> [!Important]
> Para reducir la cantidad de errores producidos por la falta del ID, se quitó el código de cada sección del CA que hacía el sync a Credentialing. Ahora solo se hace el sync cuando se hace submit del Credentialing Application.

Para solucionar este problema hay que cambiar la sincronización de IDs y de Credentialing para que la segunda solo se haga si hay el respectivo ID esperado.

## Datos

**Contactos afectados**

Kim
Contact: 237058651
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33545759309
ID: `6f00733c-2dbe-4979-a3ce-68166ac0661e`

Pivnik
Contact: 148903637833
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33605254701
ID: `ca9d3b27-7f64-4bd7-acee-d12235ae45ee`

Wang
Contact: 147350951031
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33654944316
ID: `8cad6ec4-b63c-4e9a-bd30-3b0ad663e6eb`

Schiavo
Contact: 150849614707
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33759841352/
ID: `df696364-e3ed-423f-ae63-f0b7159576c0`

> [!Note]
> Para todos estos contactos agregué manualmente la URL del AF para que el equipo de Success pueda hacer un trigger manual del sync desde ese form.


**Sentries**

Hubo varios reportes de "Missing definitive_hubspot_credentialing_id for Therapist" desde hace un día (en el momento que escribo esto) hasta hace unos poco minutos.

