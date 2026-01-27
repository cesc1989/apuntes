# EDG-3305 Patient Signed ToS PDF Outdated in backend

Etiquetas: #luna_help_desk 

## Reporte

Los pacientes veían unos términos actualizados en la app (son los términos que muestra el frontend de patient forms) pero el PDF generado que se ve en Luxe tiene los desactualizados.

En el cliente de forms: Effective Date 10/23/2025

En backend: Effective Date 3/28/2022

## Contexto

En el frontend, los términos son leídos mediante el API que está en marketing-site. De esta forma los updates que haga el equipo de Luna se reflejan enseguida.

Sin embargo, en el backend, por la forma en que se generan los PDFs, toca tener los términos en un archivo HTML. Estos no fueron actualizados así que había dos versiones. La actualizada que veían los pacientes. La desactualizada que genera el PDF con la firma.

La API es: https://github.com/lunacare/marketing-site/blob/omega/app/Http/Controllers/Web/GenericPageTextController.php#L30-L49

En estas páginas se ven los ToS y Privacy al día para prod:
- ToS: https://www.getluna.com/api/terms-of-use
- Privacy Policy: https://www.getluna.com/api/privacy-policy

# Solución

## Para los términos en las vistas

Hacer que Claudio bajé los contenidos de la API y actualicé los archivos html.erb y pdf.erb de las vistas correspondientes.

### Pacientes Alpha 🟢

Enero 2026
```
https://luxe.alpha.getluna.com/admin/patients/b96f1d87-19b2-4410-89f4-f7cf157dc4c8
https://luxe.alpha.getluna.com/admin/patients/b8f52480-d09b-48d9-ab42-4b23a59bfd53
https://luxe.alpha.getluna.com/admin/patients/0138c282-1ef5-4754-9236-852af3c32e49
```

Octubre 2025
```
https://luxe.alpha.getluna.com/admin/patients/11b2c138-7ea1-425d-8b10-5a997ca9c1c2
https://luxe.alpha.getluna.com/admin/patients/09428e34-929a-48cf-a712-a3e8dbdbb0f1
https://luxe.alpha.getluna.com/admin/patients/69f06f00-6768-498e-8e87-09be301a5c52
```

### Pacientes Omega

Enero 2026
```
https://luxe.getluna.com/admin/patients/b11e7173-a487-481f-9b80-d6aab3d6d800
https://luxe.getluna.com/admin/patients/6c7075de-a347-4e5b-b5b9-844d6470a016
https://luxe.getluna.com/admin/patients/28395c73-5f4e-4a4d-a7c6-445feae68e1a
```

Octubre 2025
```
https://luxe.getluna.com/admin/patients/97fe06a4-d902-49d5-a3f7-e548e9b74f82
https://luxe.getluna.com/admin/patients/d1b5d39e-8952-4b38-a070-0dd75c17dfcd
https://luxe.getluna.com/admin/patients/3d1b65f4-c0e6-433d-bdd6-949ffb11f0b0
```

## Para el backfill

Hay que actualizar los PDFs generados para todos esos pacientes que completaron Intake Forms después de la fecha de actualización.

Dado a que son miles de pacientes este se hará mediante un background job. Usará Sidekiq Iterable para mejor rendimiento y continuidad.

### Job del Backfill

Comando:
```ruby
PatientSelfReport::RegenerateSignedTermsPdfWorker.perform_async("2025-10-23", "2026-01-25")
```

