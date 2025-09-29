# Envío de Reminders de Appointments

La lógica para enviar estos recordatorios está definida en Marketplace _pero_ el envío del correo/sms pasa en backend.

## Definiciones

### Controlador

El controlador en backend que hospeda el código es `Api::V1::Internal::PatientCommunicationsController`. Ahí se encuentra la función `send_appointment_reminder`.

Este controlador recibe los parámetros y encola el Mailer. Desde el mailer es que se define que plantilla de correo usar.

### Mailer

El mailer es `PatientMailer`. El método en uso es `appointment_reminder`.

> [!Tip]
> Este mailer tiene su clase de preview en `PatientMailerPreview`.i

Aquí el mailer usa los parámetros para decidir que plantilla usar de esta forma:

- Si `form_type` es intake, se usa la plantilla `appointment_reminder_plus_intake`.
- Si `form_type` es progress, se usa la plantilla `appointment_reminder_plus_progress`
- Si es ninguno, se usa la plantilla `appointment_reminder`

Estos son los textos de cada plantilla.

Con Intake Form link:
![[02.intake.reminder.png]]

Vista: `app/views/patient_mailer/appointment_reminder_plus_intake.erb`

Con Progress Form link:
![[03.progress.reminder.png]]

Vista: `app/views/patient_mailer/appointment_reminder_plus_progress.erb`

Default sin link:
![[01.appointment.reminder.png]]

Vista: `app/views/patient_mailer/appointment_reminder.html.erb`

> [!Tip]
> El asunto de estos mails es "Luna Visit Reminder".

## Lógica de Envío de Recordatorio

Presente en el repo Marketplace. El flujo para enviar recordatorios está organizado en varias funciones, clases y constantes.

Hay un trabajo periodico que corre cada 15 minutos. Aquí es donde todo inicia. Está definido en `app/marketplace/rq_scheduler.py`. Función `configure_periodic_jobs`:
```python
# Send reminders to patients
scheduler.cron(_CRON_SCHEDULE_EVERY_15_MINUTES, func=communications_tasks.process_communications, use_local_timezone=True, timeout=850)
```

Adicional, la clase `CommunicationsSettings` define unas constantes con valores clave de este proceso:
```python
@attr.s(slots=True, auto_attribs=True)
class CommunicationsSettings:
    APPOINTMENT_REMINDER_HOUR: int = 48
    APPOINTMENT_REMINDER_MINIMUM_HOURS_AFTER_FORM_REQUEST: int = 24
```

La primera constante se usa para definir si el recordatorio cae entre el rango de 48 horas. Un reminder debe enviarse dentro de las 48 horas.

Usado así:
```python
# appointment not in the next X hours
if appointment.scheduled_time > now.add(hours=communications_settings.appointment_reminder_hour):
    continue
```

Finalmente, es en `app/marketplace/services/edge.py` donde está la función `send_appointment_reminder` la cual hace la petición a Edge enviando los parámetros necesarios para que se envíe el email y sms al paciente.

### Communications Jobs

En `app/marketplace/communications/jobs.py` están las funciones que hacen todo el proceso.

Estas son las funciones que se invocan y las que esas invocan a su vez:
- `process_communications_for_patients`
	- `_process_communications`
		- `get_email_events`
			- `get_patient_care_plan_email_events`
		- `lock_care_plan_and_send_patient_communication`

Es en `get_patient_care_plan_email_events` donde se recopila los datos necesarios para hacer la petición a Edge. Este es el meollo del asunto.

### Communication Tasks

Antes de llegar aquí, en `app/marketplace/communications/tasks.py` es donde está la función invocada por el trabajo de cada 15 minutos. Las funciones se ejecutan así:

- `process_communications`
	- `_process_communications`

Es en esta última donde se invoca a `jobs.process_communications_for_patients` (la función de arriba) e inicia todo el proceso.