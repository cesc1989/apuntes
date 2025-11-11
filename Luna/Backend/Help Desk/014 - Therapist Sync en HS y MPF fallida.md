# 014 - Sync a HubSpot y MPF de Therapist Fallido

etiquetas: #luna_help_desk 

Caso EDG-2991

Por los cambios que ha estado haciendo Anthony para el AppBlend se dieron varios errores en los syncs de Therapist cuando se registran. En este reporte los errores fueron un fallo de sinc de propiedades en HubSpot y la generación de las carpetas del MPF.

Lo del MPF no tiene nada que ver pero pasó para este therapist afectado.

# Solución

Hay que agendar los jobs para hacer los syncs en ambos sistemas externos.

## Agendar jobs para sync a HubSpot 🟢

La primera parte es reagendar los jobs para actualizar las propiedades en HubSpot:
```ruby
Credentialing::HubspotFormFilesWorker.perform_async("034dcade-119c-4683-b394-0d09f35b9015")
Credentialing::HubspotFormUrlWorker.perform_async("034dcade-119c-4683-b394-0d09f35b9015")
Credentialing::SignupPdfWorker.perform_async("034dcade-119c-4683-b394-0d09f35b9015", "independent_contractor")
```

Estos tres workers son para actualizar las propiedades:

- Credentialing Form Files
- Credentialing Form URL
- Credentialing Signup PDF

## Retrigger sync de S3 a GDrive

Hay un comando en Marketplace para hacer un retrigger cuando hay fallos al sincronizar entre S3 y GDrive.

Datos para preparar el comando:

- Creación del therapist: 11/06/2025
- ID: 034dcade-119c-4683-b394-0d09f35b9015

Comando:
```bash
python -m marketplace.commands.google_drive retrigger-s3-onboarding-files \
    --bucket luna-omega-workloads-therapist-credentialing \
    --days-back 7 \
    --prefix "034dcade-119c-4683-b394-0d09f35b9015/"
```

### Prueba en Local

> [!Warning]
> No se puede probar en local porque lo que hace el comando es redisparar la lambda en S3. Al dispararse la lambda se manda una petición al servidor de Marketplace. No tengo forma de lograr esto en local.

Comando:
```bash
python -m marketplace.commands.google_drive retrigger-s3-onboarding-files \
    --bucket luna-alpha-workloads-therapist-signup \
    --days-back 5 \
    --prefix "6bf5d3fe-63f1-4c17-ac56-689e9ac5c783/"
```