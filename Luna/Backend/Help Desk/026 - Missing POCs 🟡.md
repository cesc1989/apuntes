# 026 - Missing POCs

Etiquetas: #luna_help_desk 

Caso: EDG-1309

Sub casos:
- EDG-3135: agregar logs 🟢
- EDG-3137: probar generación en retry 🟡

## Contexto

Se reporta que desde Enero ha estado fallando la creación de POCs. Hubo 825 casos en Enero. A Diciembre sigue habiendo casos.

Todo empieza en `TherapistSignedChartPdfGeneratorWorker`. Este worker hace varias cosas. 

Comentario de Ryan:
> Then you need to look at the state files to see the rulset that applies, e.g. `georgia.yml` / `plan_of_care_rules_config`.
>
> Then with the referral type you would look up the appropriate POC message calculator
>
> Summary:
>
> - Therapist signature triggers POC generation  
> - Each referral type has a separate file that implements the logic
> - If we have a bug, it is likely in one of those files.

# Tareas

## Agregar logs en puntos clave

Para poder inspeccionar y tener datos extra de cómo se da este proceso.

