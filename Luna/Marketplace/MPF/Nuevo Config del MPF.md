# Nuevo Config del MPF

Por los múltiples problemas que Marketplace encontraba con la configuración de varias Service Accounts llegando al límite de archivos, Ryan decidió cambiar la forma en que el MPF funciona. Aquí trataré de ir describiendo todo lo nuevo.

## Legacy

La env `GOOGLE_DRIVE_ROOT_ONBOARDING_FOLDER_ID` es legacy y ya no se usa en la nueva configuración.

Se usa activamente en `app/marketplace/onboarding/jobs.py`:
```python
# Legacy folder - kept for backward compatibility during migration
ROOT_CREDENTIAL_FOLDER = settings.GOOGLE.GOOGLE_DRIVE_ROOT_ONBOARDING_FOLDER_ID
```

Pero al buscar la constante `ROOT_CREDENTIAL_FOLDER` en el archivo no hay resultados. O sea que ya no está en uso.

## Arquitectura de 4 carpetas


### Crear nuevas carpetas y Compartir con email del Service Account

Hay que crear 4 carpetas a mismo nivel y todas deben ser compartidas con el mismo email de la Service Account.

El email es:
```
nalufilesmagic@nalu2023.iam.gserviceaccount.com
```

Las carpetas deberían llamarse:
