# 012 - Therapist Boosted está contando Waitlist self-selected

Etiquetas: #luna_help_desk 

Caso EDG-2899

Reporte:

> Boosted Therapist feature counting self selected Waitlist patients from therapists as organic bookings

# Contexto

Cuando un therapist está con pocos pacientes para atender puede activar una opción para tomar los pacientes de "Waitlist". Además, cuando están cortos de pacientes desde Luna está la opción de darles un "boost" de manera temporal para darles prioridad en la recepción de pacientes.

Cuando se "boostea" se defina una cantidad de pacientes que quieren ver antes de quitar el "boost".

La información sobre el estado de Waitlist y la disponibilidad del paciente se muestra en una caja en la parte superior derecha del perfil.

![[patient.waitlist.status.png]]

Nota como hay dos botones:

- Edit Waitlist Availability
- Manage Waitlist Status

Es porque ambos manipulan modelos diferentes.

## Waitlist status

Se define en el modelo `WaitlistEntry`.

## Availabilities

Se define en el modelo `Availability`.

# Problema

Cuando el therapist es "boosted" y además también activó la opción de tomar pacientes del "Waitlist" el sistema está contando ese "Waitlist" como si Luna lo hubiera agendado. Lo cual es incorrecto.

En el reporte dice:
> Waitlist patient selections should not apply to this count.

# Replicar Error

## Therapist Boost

Hay que crear un "Therapist Boost" para cualquier therapist. Esos se crean yendo a http://localhost:3000/admin/therapist_boosts/new

> [!Note]
> La página es MUY lenta para cargar. Recomiendo usar el 1er therapist que cargue.

## Paciente en Waitlist

