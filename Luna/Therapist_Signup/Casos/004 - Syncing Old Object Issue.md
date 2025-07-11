# 004 - Sincronizando con HS old objects (Inactive Cred)

Una vez un objeto Credentialing toma la label "Inactive" no debería recibir más actualizaciones.

Jose reporta esto:

> With DOA therapist I ==_signed up again_==, then ==_submitted AF_== but it updated the old treating license that was associated with the old Credentialing object. Issue is the integration updated the Treating license of the 'Inactive' label credentialing object. It shouldn't update the treating license.  
>
> Treating license that got updated by CA/AF: [https://app.hubspot.com/contacts/4634981/record/2-35450678/30547332684](https://app.hubspot.com/contacts/4634981/record/2-35450678/30547332684)  
>
> Other issue: It appears the ==**additional licenses didn't get created in this new Credentialing object after submitting CA/AF**==: [https://app.hubspot.com/contacts/4634981/record/2-31374266/30528403996](https://app.hubspot.com/contacts/4634981/record/2-31374266/30528403996)

Aquí hay dos problemas que están relacionados:

- Se actualizó el objeto Credentialing Inactive
- No se crearon las additional Licenses (existentes para el Cred Inactive) para el nuevo Cred Active Attested

## Additional Licenses no se transfieren al nuevo Credentialing Active Attested

Sospecho que pasa que no se transfieren a un nuevo objeto Active Attested. Ejemplo:

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

