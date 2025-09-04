# 006 - Credentialing object no recibe sync desde CA

Etiquetas: #luna_help_desk 

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

¿Tiene ya los IDs?
```bash
credentialing_active_attested_id: 33545759309
credentialing_relocation_id: 33534500134
```

En Logs
```bash
2025-08-25 19:13:08.065	
I, [2025-08-26T00:13:08.065521 #1]  INFO -- : Did not found Associated Custom Objects for Hubspot ID: 237058651
```

Registro del log: 2025-08-25 19:13:08
Credentialing creado: 08/25/2025 7:14 PM GMT-5

---

Pivnik
Contact: 148903637833
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33605254701
ID: `ca9d3b27-7f64-4bd7-acee-d12235ae45ee`

¿Tiene ya los IDs?
```bash
credentialing_active_attested_id: 33605254701
credentialing_relocation_id: 33606029484
```

En Logs
```bash
2025-08-26 19:46:35.944	
I, [2025-08-27T00:46:35.944573 #1]  INFO -- : Did not found Associated Custom Objects for Hubspot ID: 148903637833
```

Registro del log: 2025-08-26 19:46:35
Credentialing creado: 08/26/2025 7:47 PM GMT-5

---

Wang
Contact: 147350951031
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33654944316
ID: `8cad6ec4-b63c-4e9a-bd30-3b0ad663e6eb`

¿Tiene ya los IDs?
```bash
credentialing_active_attested_id: 33654944316
credentialing_relocation_id: 33657099651
```

En Logs
```bash
2025-08-28 08:06:48.933	
I, [2025-08-28T13:06:48.933469 #1]  INFO -- : Did not found Associated Custom Objects for Hubspot ID: 147350951031
```

Registro del log: 2025-08-28 08:06:48
Credentialing creado: 08/28/2025 8:08 AM GMT-5

---

Schiavo
Contact: 150849614707
Credentialing: https://app.hubspot.com/contacts/4634981/record/2-31374266/33759841352/
ID: `df696364-e3ed-423f-ae63-f0b7159576c0`

¿Tiene ya los IDs?
```bash
credentialing_active_attested_id: 33759841352
credentialing_relocation_id: 33748064656
```

En Logs
```bash
2025-08-28 23:01:29.560	
I, [2025-08-29T04:01:29.560179 #1]  INFO -- : Did not found Associated Custom Objects for Hubspot ID: 150849614707
```

Registro del log: 2025-08-28 23:01:29
Credentialing creado: 08/28/2025 11:02 PM GMT-5

---

> [!Note]
> Para todos estos contactos agregué manualmente la URL del AF para que el equipo de Success pueda hacer un trigger manual del sync desde ese form.

Query en Grafana:
```sql
{app="therapist-credentialing-backend"} |= `Did not found Associated Custom Objects for Hubspot ID: 150849614707`
```

**Sentries**

Hubo varios reportes de "Missing definitive_hubspot_credentialing_id for Therapist" desde hace un día (en el momento que escribo esto) hasta hace unos poco minutos.

# Solución

Para Credentialing object.

En el worker `HubspotSyncCustomObjectIdsToTherapistWorker` se debe tener un resultado de la ejecución del servicio `HubspotSyncCustomObjectsIdsToTherapistService`. Un `true` o `false`.

Si es exitoso el servicio, entonces se encola el worker `HubspotCustomObjects::HubspotCredentialingObjectWorker`.

> [!Note]
> Si esto funciona para Credentialing, lo llevaré para License.

## Pasos

Pasos para completar la solución:

- [x] Usar `Hubspot::Connection` para simplificar el servicio
- [x] Envolver `Hubspot::Connection` en un bloque `retriable`
- [x] Usar la respuesta del servicio para encolar el worker de `HubspotCredentialingObjectWorker`
- [x] Usar la respuesta del servicio para encolar el worker de `HubspotLicenseObjectWorker`
- [x] Escribir pruebas automáticas
- [x] Probar en local
- [x] Probar en alpha multiples sign ups
- [x] Probar en alpha multiples ESUs

# Resumen de Actualizaciones

Summary of updates introduced by Linear card.

> Note: **EDG-2618** acts as parent ticket for the following.

**EDG-2617 - Remove old feature flag**: This was cleanup job before starting the required changes.

**EDG-2621 - Only sync to Credentialing object after CA is submitted**: Previously, after every section the system would try to sync. That would fail if the required ID was not present. Here we made the change to only sync after the Credentialing Application is submitted so that this is the only point of failure.

**EDG-2623 - Improve query to pull Custom Object IDs from HubSpot**: Improve the query to only pull IDs for those records without an `credentialing_active_attested_id` in the DB. This lowered the amount of records to update and requests to make to HubSpot.

**EDG-2624 - Remove batch sync of custom object IDs**: After improving the query I tracked results and turns out this was being wasteful. The list of therapists to pull IDs to was not reducing in important numbers.

**EDG-2625 - Pull object IDs after completing each CA section**: Instead of an hourly batch pull now the system pulls the ID if missing after completing each section of the Credentialing Application.

**EDG-2618 - Improve sync of Custom Object IDs**: Technical changes in the service that pulls IDs from HubSpot. Simplify code and made it return a success/failure indicator to use in the upcoming retry setup.

**EDG-2632 - Only sync to Credentialing object afterAF is submitted**: Same as in EDG-2621 but applied to the Attestation Form.

**EDG-2633 - Pull object IDs after completing each AF section**: same as in EDG-2625 but applied to the Attestation Form.