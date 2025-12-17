# 026 - Missing POCs

Etiquetas: #luna_help_desk 

Caso: EDG-1309

Sub casos:
- EDG-3135: agregar logs 🟢
- EDG-3137: probar generación en retry 🟡

## Contexto

Se reporta que desde Enero ha estado fallando la creación de POCs. Hubo 825 casos en Enero. A Diciembre sigue habiendo casos.

Todo empieza en `TherapistSignedChartPdfGeneratorWorker`. Este worker hace varias cosas. Ver [[POC Generation - Missing Case Analysis#Entry Point TherapistSignedChartPdfGeneratorWorker]]

Comentario de Ryan:
> Then you need to look at the state files to see the ruleset that applies, e.g. `georgia.yml` / `plan_of_care_rules_config`.
>
> Then with the referral type you would look up the appropriate POC message calculator
>
> Summary:
>
> - Therapist signature triggers POC generation  
> - Each referral type has a separate file that implements the logic
> - If we have a bug, it is likely in one of those files.

# Tareas

## Agregar logs en puntos clave 🟢

Para poder inspeccionar y tener datos extra de cómo se da este proceso de tal forma que ayude a ver donde se salta la generación y tener idea de por qué.

## Probar extraer generación del POC de generación del PDF ❌

> Como se explica en [[POC Generation - Missing Case Analysis#2. CHART SIGNING STATE BUG ❌]]

Esto no es la causa.

## Comprobar si los missing POCs son de estados que no tienen configuración alguna

O si hay alguna mala configuración.

Estados que no tienen configuración de POC son los listados en [[POC Generation - Missing Case Analysis#🟡 States WITHOUT POC Config (10 states) 🟡]]
