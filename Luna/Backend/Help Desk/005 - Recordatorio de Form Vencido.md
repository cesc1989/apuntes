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

# Comprobar `completed_at` entre tablas

Hay que comprobar este valor para los siguientes forms:

**Para Fenn:**

Julio 21 al 22. Form `e5f30c55-3122-445c-af12-744fe14b1748`

- Edge `completed_at`: 2025-07-15 16:45:00.597
- Marketplace `completed_at`: nulo

Julio 23. Form `b5f1a48a-6903-4a4e-a987-3b9c7a1af32f`

- Edge `completed_at`: 2025-07-31 17:18:55.238
- Marketplace `completed_at`: nulo

**Para Sohn**

Julio 22. Form `536dad03-95df-4d44-a673-7215898b7975`

- Edge `completed_at`: 2025-07-05 14:49:04.798
- Marketplace `completed_at`: nulo

## Conteo de Progress sin `completed_at` entre Backend y Marketplace

Dado lo anterior, corrí esta query para saber cuántos progress forms no tienen el campo `completed_at`. La comparación entre ambos sistemas es MUY grande.

**backend**

|form_type|total_forms|with_completed_at|without_completed_at|percent_complete|
|---------|-----------|-----------------|--------------------|----------------|
|ongoing|360188|258357|101831|71.73|
|onboarding|236329|167638|68691|70.93|

**marketplace**

|form_type|total_forms|with_completed_at|without_completed_at|percent_complete|
|---------|-----------|-----------------|--------------------|----------------|
|progress|359513|166193|193320|46.23|
|intake|234843|165893|68950|70.64|

Esto dice Claudio:

### Analysis

**Intake Forms (Good Sync)**

- Backend: 70.93% completed
- Marketplace: 70.64% completed
- Difference: Only 0.3% - nearly perfect sync ✅

**Progress Forms (Broken Sync)**

- Backend: 71.73% completed
- Marketplace: 46.23% completed
- Difference: 25.5% gap - massive data loss ❌

**What This Means**

~92,000 progress forms are missing `completed_at` in Marketplace (258,357 - 166,193)

# El Problema

Es que en 2023 cuando se corrigió un problema del worker de webhook de form completed no agregué el código para cuando se completan los Progress Forms. Ver commits:

- Junio 28, 2023: se introduce el webhook
	- Commit: https://github.com/lunacare/patient-forms-backend/commit/a44b403fba50eb6e5092c19209ed17e917ada2fa
- Junio 30, 2023: Ryan corrige un problema de asincronía sin terminar la transacción que completa el Form
	- Commit: https://github.com/lunacare/patient-forms-backend/commit/298f3047535cd4eea35458f82f52fe8867a177a4
- Julio 4, 2023: Saco el worker del modelo y lo pongo solo en la clase `AdmissionForm`.
	- **Aquí se introdujo el error.**
	- Esta es la clase que completa el Intake Form.
	- Commit: https://github.com/lunacare/patient-forms-backend/commit/d06df88b0f66deea2ca3f13936e5658890552c4a

> [!Note]
> Historial de commits del modelo Form: https://github.com/lunacare/patient-forms-backend/commits/omega/app/models/form.rb