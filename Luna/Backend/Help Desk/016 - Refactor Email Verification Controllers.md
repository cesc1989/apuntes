# 016 - Refactor Email Verification Controllers

Etiquetas: #luna_help_desk 

Caso EDG 2743

Reporte

A causa de un error donde pacientes estaban siendo redireccionados a la página de success de verificación de email para los usuarios de Clinical Dashboard Fabricio optó, para arreglarlo, por duplicar el controlador. La diferencia terminó siendo mínima pero fue necesario para arreglar el lío.

Como parte de las tareas salidas del Postmortem está refactorizar estos controladores para simplificar el código y limpiar lo que se hizo para dejar todo en mejor forma.

## Contexto

Los controladores son:

- `app/controllers/user_communication_methods/email_verifications_controller.rb`
- `app/controllers/user_communication_methods/patient_email_verifications_controller.rb`

## Solución

Un controlador base que tenga toda la lógica general y controladores hijos que hagan las diferencias.

## Pruebas

Para más detalles ver:

- [[Probando Email Verification Landing]]
- [[Probando Salida de Correos en Local]]

Detalles para este caso a continuación.