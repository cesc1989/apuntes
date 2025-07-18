# 004 - Sincronizando con HS old objects (Inactive Credentialing)

Una vez un objeto Credentialing toma la label "Inactive" no debería recibir más actualizaciones.

Jose reporta esto:

> With DOA therapist I ==_signed up again_==, then ==_submitted AF_== but it updated the old treating license that was associated with the old Credentialing object. Issue is the integration updated the Treating license of the 'Inactive' label credentialing object. It shouldn't update the treating license.  
>
> Treating license that got updated by CA/AF: [https://app.hubspot.com/contacts/4634981/record/2-35450678/30547332684](https://app.hubspot.com/contacts/4634981/record/2-35450678/30547332684)  
>
> Other issue: It appears the ==**additional licenses didn't get created in this new Credentialing object after submitting CA/AF**==: [https://app.hubspot.com/contacts/4634981/record/2-31374266/30528403996](https://app.hubspot.com/contacts/4634981/record/2-31374266/30528403996)

Aquí hay dos problemas que están relacionados:

- No se crearon las additional Licenses (existentes para el Cred Inactive) para el nuevo Cred Active Attested
- Se actualizó el objeto Credentialing Inactive

Veamos.

## Additional Licenses no se transfieren al nuevo Credentialing Active Attested

Sospecho que pasa que no se transfieren a un nuevo objeto Active Attested.

Ejemplo de un solo Credentialing Active Attested.

1. Cred Active Attested existe
2. Usuario ingresa License en otro estados
3. CA submitted: se crean las Additional Licenses
4. AF actualizado: se actualizan
5. AF submitted: no pasa nada

¿Qué pasa si vuelve a hacer sign up?

1. Cred Active Attested nuevo
2. Anterior pasa a Inactive
3. AF actualizado: se actualizan los Licenses (tienen su propio `hubspot_id`)
4. AF submitted: el nuevo Cred no tiene los Additional Licenses

> [!Note]
> El objeto Credentialing sí es Inactive. En el caso que reporta Jose el Contact tiene un objeto Inactive y otro Active Attesting.
>
> Esa es la clave para el fix. Si el campo `credentialing_inactive_hubspot_id` tiene valor, entonces hay que reasociar esas Additional Licenses si las tiene.

Creo que esto es el problema.

Lo que postee en Linear:

> The issue is for 2nd sign ups.
> 
> After a Returning/Moving sign up, a new Credentialing Active Attested is created. When this happens, usually, the therapist has already completed their Credentialing Application. In this case, any existing Additional License is linked to the _previous_ (now _Inactive_) Credentialing object.
> 
> So when a therapist completes their AF and there's information in the "licenses in other states" section, those Additional Licenses are not created for the new Active Attested Credentialing object.
> 
> In the end is something like this:
> 
>  - License in other state: California, PT1234
>  - Inactive Credentialing -> Additional License (California)
>  - Active Attested Credentialing -> 😞

### ¿Qué lo explica?

Donde se usan las clases que crean y actualizan Additional Licenses.

El worker (`HubspotCreateLicenseInOtherStateWorker`) que se ejecuta después de las peticiones:

  1. Definition: `app/workers/hubspot_custom_objects/hubspot_create_license_in_other_state_worker.rb:2`
  2. Usage: `app/controllers/api/v1/submit_controller.rb:48` - scheduled to run in 5 minutes with "create" action
  3. Usage: `app/controllers/api/v2/external/attestation/credentialing_informations_controller.rb:119` - runs async with "update" action
  4. Tests: Referenced in specs for both controller endpoints

La clase(`HubspotLicenseInOtherStateService`) que hace el trabajo:

  1. Definition: `app/services/hubspot_custom_objects/hubspot_license_in_other_state_service.rb:9`
  2. Usage: `app/workers/hubspot_custom_objects/hubspot_create_license_in_other_state_worker.rb:10` - for creating records
  3. Usage: `app/workers/hubspot_custom_objects/hubspot_create_license_in_other_state_worker.rb:14` - for updating records

### Solución

Hay dos opciones:

- Crear nuevos Additional License para cada Active Attested
	- De esta forma el Inactive conserva los suyos
- Re-asociar los Additional Licenses del Inactive al Active Attested ✅

Lo segundo fue la opción indicada por Jose Han.

> [!Important]
> La solución está implementada y en pruebas. Todo el estado está en [EDG-2412](https://linear.app/getluna/issue/EDG-2412/fix-additional-license-objects-not-being-created-after-af-submission).

