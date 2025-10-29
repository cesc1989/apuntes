# 012 - Therapist Boosted está contando Waitlist self-selected

Etiquetas: #luna_help_desk 

Caso EDG-2899

Reporte:

> Boosted Therapist feature counting self selected Waitlist patients from therapists as organic bookings

## Contexto

Cuando un therapist está con pocos pacientes para atender puede activar una opción para tomar los pacientes de "Waitlist". Además, cuando están cortos de pacientes desde Luna está la opción de darles un "boost" de manera temporal para darles prioridad en la recepción de pacientes.

Cuando se "boostea" se defina una cantidad de pacientes que quieren ver antes de quitar el "boost".

## Problema

Cuando el therapist es "boosted" y además también activó la opción de tomar pacientes del "Waitlist" el sistema está contando ese "Waitlist" como si Luna lo hubiera agendado. Lo cual es incorrecto.

En el reporte dice:
> Waitlist patient selections should not apply to this count.