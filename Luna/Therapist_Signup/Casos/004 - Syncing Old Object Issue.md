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

