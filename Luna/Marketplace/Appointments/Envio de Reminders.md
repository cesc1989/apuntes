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

Con Progress Form link:
![[03.progress.reminder.png]]

Default sin link:
![[01.appointment.reminder.png]]