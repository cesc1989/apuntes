# Apuntes del Onboarding en OrderlyMeds

Apuntes de la reunión con Jaime y Fabian.

> [!Important]
> Todos los CX existen en Ontraport. Hay una migración activa de Ontraport a Salesforce. Esto significa que no todos los cx existen en Salesforce.

> [!Info]
> Muchos casos se resuelven por consola de Heroku. O sea, modificando datos o disparando jobs.
>
> Desde el perfil del Customer en Salesforce también se pueden dispara tareas de corrección.

## Sobre Member Period

- El equipo de Customer Support lo llama MP.
- Solo existen en Salesforce.


## Sobre Script

- Solo existe en Ontraport.

### Casos de Stuck In / Script error

Reportes de algo "stuck in" o "script" son de problemas en Ontraport.

#### Para los casos de Stuck In

Copiar el correo y buscarlo en la sección "Case Overview". En la pestaña "Care Validate Submissions" se pueden ver los estados del flujo.

##### Si el estado es `waiting_for_prescription`
> [!warning]
> Jaime hizo una grabación de este paso. Ver y tomar apuntes de eso.

- haz clic en el ID linkeado a la vista "Care Validate Request"
	- Ahí se puede ver todo el flujo.
	- Si no hay nada extraño, indicar al Rep que debe contactar con Care Validate.


##### Si el estado es `routed_to_beluga`

> [!warning]
> Jaime hizo una grabación de este paso. Ver y tomar apuntes de eso.

Es otro caso. Hay que revisar el flujo del state machine y los datos en la base de datos.
