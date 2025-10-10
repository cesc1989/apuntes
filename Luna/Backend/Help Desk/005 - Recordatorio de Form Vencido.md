# 005 - Pacientes reciben recordatorio de Form vencido o por crear

Etiquetas: #luna_help_desk 

Caso EDG-2534.

Reporte:
> I have 2 patient reports that they are receiving visit reminder emails with incorrect progress form notifications. ==_These emails say that the form is 'required' before next visit, but the form has not yet been created in luxe==, and therefore the email's hyperlink to the form does not work_ (links to a page that says the progress form is already complete).

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

Claudio armó esta línea de tiempo:

1. **8:30 AM**: 4th appointment starts (patient on their way to 5th appointment)
2. **9:25 AM**: 4th appointment completes
3. **9:25 AM**: Backend creates progress form immediately
4. **9:26 AM**: Marketplace reminder scheduler runs (every 15 minutes)
5. **9:26 AM**: Marketplace queries `latest_progress_form` at line 162
6. ~~**9:26 AM**: Marketplace database **hasn't synced yet** - sees OLD completed form or NO form~~
	1. Esto es incorrecto porque las tablas de forms no tienen Logical Replication.
7. **9:26 AM**: Check at line 302 evaluates against stale data:
   - If sees completed form: Falls through to line 305 (no form link)
   - If sees no form: Falls through to line 305 (no form link)
8. **9:26 AM**: Email sent **without progress form link** 📧❌
9. ~~**Later**: Database sync completes, but email already sent~~
	1. No estoy convencido de esto.

Esta línea da una pista e ida pero no es del todo correcta:

- No hay Logical Replication entre `marketplace.patient_form` y `edge.forms`
- Cuando Marketplace pide crear un Form en Edge, este se crea en Edge y en Marketplace
- La query para `latest_progress_form` devuelve forms incompletos por defecto por lo que sí podría pasar que se encuentra un form ya completo pero:
	- El código de preparación para el recordatorio hace un check contra la propiedad `completed_at` del Form
	- O sea que para que mande el recordatorio con link viejo este check debe estar fallando

## Cómo Detectar el Problema

Claudio sugiere estas formas para detectar el problema:

1. Poner logs en la parte donde se busca `latest_progress_form`
2. Comparar los datos en las tablas entre backend y Marketplace
	1. Si no existe el form en Marketplace, entonces hay un problema de sincronización
3. Comprueba el sync de los datos
	1. Compara ambas tablas prestando atención a `completed_at`
	2. Si no son iguales para un mismo form, por aquí podría ser el problema


Los logs me parece muy lento porque toca esperar a que se refleje. Creo que puedo pasar directo a hacer queries en las BDs.