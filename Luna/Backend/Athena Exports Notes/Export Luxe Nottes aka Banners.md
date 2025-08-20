# Export de Luxe Notes o Banners para Tableau

Solicitaron que se exportaran a Tableau los datos que aparecen en los banners del perfil del paciente (los open issues).

![[patient.open.issues.png]]

Estos banners incluyen comentarios que se hacen en el perfil del paciente o `collection_phase_warnings`.

Así está expresado en código:
```ruby
collection_phase_warnings =
  if resource.collections_status&.past_due?
    phase_warning =
      "Patient has an outstanding past due balance. If patient contacts Luna, transfer to Billing."
    if resource.collections_status.balance_past_due_at.present? # nil possible during QA setup
      formatted_past_due_date = resource.collections_status.balance_past_due_at.to_date.to_fs
      phase_warning = "#{phase_warning} (Balance marked past due on #{formatted_past_due_date})"
    end
    [phase_warning]
  else
    []
  end

open_issues = @patient.open_issues + collection_phase_warnings

if open_issues.present?
  br
  table_for open_issues, class: "patient_alert", style: "width: 100%;" do
    column "Open Issues" do |open_issue_alert|
      ul style: "margin-block-start: 0; margin-block-end: 0; padding-left: 24px;" do
        li do
          open_issue_alert
        end
      end
    end
  end
end
```

## Tablas involucradas

Para lograr este export armé varios CTE en `Athena::PatientSummaryWriterWorker`.

Se lograron usando estas tablas:

- `active_admin_comments` para lo que se publica desde el perfil del paciente
- `patient_collections_statuses` lo que respecta a las fases

## Athena: crawler, classifier y table en Alpha

Para probar este export en Alpha me tocó seguir el runbook para crear una tabla a partir de un crawler. No había nada prácticamente.

# Reversión por error de campo

Explotó el export en Omega porque el campo de `active_admin_comments` puede tener saltos de línea y otras cosas. Entonces el export en Athena se veía así:

![[broken.export.png]]

Para reverse tuve que:

- Hacer PR de revert
- Esperar que el despliegue completara
- Pedir ejecutar el worker `Athena::PatientSummaryWriterWorker.perform_async`
- Esperar a que complete el worker
- Revisar en S3 los archivos data.csv de cada partición se actualizaran
- Correr de nuevo el Crawler "Business Operations - Patient Summary"
- Revisar Athena

## Casos Especiales

**El worker le puede tomar bastante tiempo lograr el export en todas las particiones de la carpeta de cada tabla.**

Pasó que en la carpeta de Agosto 2025 ya no se veía la columna "open_issues" en el CSV pero sí aparecía en el export más antiguo (2018). Pasados varios minutos se actualizó para no tener más esa columna.

**Columna aún aparece en Athena porque el crawler no puede borrarla del definición.**

Por lo anterior, la columna aún aparecía. Para corregir en este caso fui a la definición de la tabla y borré la columna del esquema.