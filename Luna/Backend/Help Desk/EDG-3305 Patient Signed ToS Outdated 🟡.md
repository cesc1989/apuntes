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

## Para el backfill

Hay que actualizar los PDFs generados para todos esos pacientes que completaron Intake Forms después de la fecha de actualización.

Dado a que son miles de pacientes este se hará mediante un background job. Usará Sidekiq Iterable para mejor rendimiento y continuidad.

### Job del Backfill

Comando:
```ruby
PatientSelfReport::RegenerateSignedTermsPdfWorker.perform_async("2025-10-23", "2026-01-25")
```

