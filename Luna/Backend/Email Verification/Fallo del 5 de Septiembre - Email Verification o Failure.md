# Fallo del 5 de Septiembre: patients accediendo a página que no correspondía

El 5 de Septiembre reportaron que los pacientes al verificar su email estaban viendo la página que se espera es para usuarios del Clinical Dashboard (physicians, clinics, practices).

Ejemplo, esta página de cuando falla la verificación del email:

![[301.failed.email.verification.png]]

El arreglo lo hizo Fabricio al duplicar la ruta, controlador, vistas y separar cada landing.

## Causa

El error está en este bloque de código:
```ruby
if @communication_method&.verified?
	redirect_to verify_email_success_path(base64_token: params.require(:base64_token))
end
```

No está discriminando entre Patient o ProviderEntity.

## Replicando en Local

Con la siguiente URL se puede confirmar que las instancias de `UserCommunicationMethods` de `Accounts` de `Patients` fueron llevadas a las páginas de verificación fallida/exitosa diseñada solo para los providers que tendrían acceso al Clinical Dashboard.

URL:
```
http://localhost:3000/verify_email/eyJpZCI6IjAxYmIwNTI5LTBkMjUtNGY4NC04ODQ2LTc4NWVmMDg2ZjBkMSIsImNvZGUiOiI1UW5sMEt2ay14SUpkWDVNblByaXRnIn0=
```

Datos de esta URL:
- UserCommunicationMethod ID: `01bb0529-0d25-4f84-8846-785ef086f0d1`
- Email: `ivan.barona+faker28782@koombea.com`
- Patient ID: `2d22a50e-3131-46d1-adc3-bdc56007f0c8`
- Verification Status: verified
- Primary Method: true

