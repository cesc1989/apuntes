# 004 - Paciente Recibió Progress Form Email sin el Link

Etiquetas: #luna_help_desk 

Reporte:
> (...) form was noted as sent but no email was pushed out to patient with (...) form link.

Jordan explica:
> **Background Details**
>
> - The 4th appointment (progress form appointment) occurred on 08/11 @ 8:30 AM PDT (55 min). The 5th appointment occurred on 08/13 @8:30 AM PDT
> - *A progress form is only created after the appointment is completed*
> - An ==email reminder is sent for the next appointment approximately 48 hours prior to the upcoming appointment start time==, and the path of the email sent is only changed by the existence of the form being created and not completed.
>
> **Series of Events on 08/11 starting at 8:30 AM PDT**
> 
> - 4th appt starts at 8:30 AM PDT 
> - 48 hour window till 08/13 8:30 AM PT (5th appt) occurs and the scheduled job in backend which runs every 15 min finds the patient has an upcoming appt within next 48 hours w/o a progress form. 
> - System sends email without progress form attached as 4th appt is still ongoing 
> - 4th appointment ends and progress form is created. 
> - 5th appointment reminder occurs again and states outstanding progress form language which has expected that patient has already received communication.
>
> **Result**
> Patient did not receive email due to timing of progress appointment in relation to the upcoming appointment aligning within the 48 hour window resulting in communication not being sent.

Luego Jordan comenta:
> There is that gap of time where if its during the initial and the next appointment is during the time period where it is less than 48 hours, the therapist won't receive the appropriate communications due to that overlap of timing/appointment timing.


Resumén de Claudio:
> This is a race condition bug where the 48-hour email reminder job runs before the appointment is completed and the progress form is created, resulting in patients not receiving the form link email.