# 005 - Pacientes reciben recordatorio de Form vencido o por crear

Etiquetas: #luna_help_desk 

Caso EDG-2534.

Reporte:
> I have 2 patient reports that they are receiving visit reminder emails with incorrect progress form notifications. _These emails say that the form is 'required' before next visit, but the form has not yet been created in luxe, and therefore the email's hyperlink to the form does not work_ (links to a page that says the progress form is already complete).

Hubo un PR de Luis pero se cerró porque no atacaba el problema de raíz. Para ello Ryan sugirió:

> - If it is _not_ a one-off, as I suspect, we should either:
> 	- Fix the root cause (including backfill to ensure Marketplace/Backend form completion status are in sync) **- OR -**
> 	- Migrate reminder code to one of Backend or Grimoire.

Dado a los reportes recientes esto no es un one-off:
- July 21-22: Multiple incorrect emails, then correct one on July 23
- July 22: Incorrect notification
- August 5: Another incorrect email (form completed 7/31, email sent 8/5)
- August 13: Clicked email link, saw "form already submitted"
- September 18: Additional reports via Therapist

La segunda sugerencia de Ryan es _mucho_ trabajo así que lo descarto por ahora.

## El Problema según Claudio



## Detectar el problema

Claudio sugiere dos formas de confirmar el problema:

1. Poner logs
2. Comparar los datos en las tablas entre backend y Marketplace

