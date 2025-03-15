# Flujo de Onboarding Form y Progress Form

# Flujo del *Onboarding Form*
## 1. Crear paciente y generar enlace a formulario de *onboarding*

Endpoint: `POST /patients_forms`
Controlador: `PatientFormsController`

## 2. Al crearlo, se genera un enlace para ir al *onboarding/intake form*

Endpoint: `GET /patients/:internal_id/forms/:form_uuid`

## 3. El enlace anterior es el endpoint para ver el *onboarding/intake form* en la aplicación web.

Además, se hacen peticiones a otros endpoints para datos del formulario:

- `GET /diseases`
- `GET /questions`
- `GET /pain_spots`
## 4. Se guarda el *onboarding/intake form* en su totalidad

En la primera página del formulario hay validaciones por *frontend*. En la segunda página hay validaciones(de todo) por *backend*.

Endpoint: `POST /forms/:uuid/onboarding_forms`

Que hace lo siguiente:

- Actualiza al paciente(`Patient`)
- Crea el `IntakeForm` del *onboarding* (`Form`)
- Guarda las *aggravating_activities* del `Form`
- Guarda las respuestas `Answers`
- Actualiza la escala de dolor del `Form`
- Marca el *onboarding* `Form` como completo
    - Atributos: `completed` y `completed_at`

También se encarga de validar:

- Parámetros
    - para *form*
    - para *patient_attributes*
    - para *intake_attributes*
    - para *aggravating_activities*
    - para *answers*
- Si ya está previamente completado
- Si los términos y condiciones fueron aceptados
- Si el paciente firmó
# Flujo del Ongoing/Progress Form

Este es un formulario mucho más corto que el *onboarding* ya que es para llevar un registro del progreso o evolución del paciente en el tratamiento.

**Se compone de**:

- Escala de dolor: *pain_scale*
- Actividades agravantes: *aggravating_activities*
- Respuestas a preguntas predefinidas: *answers*
## 1. Se crea un enlace con para acceder al progress form

Endpoint: `POST /forms/:form_uuid/request_progress_forms`

Este endpoint crea un `Form` para un paciente existente(quien ya viene del *onboarding*).

**A tener en cuenta**

- Se crea con un `patient_id`
- El campo `onboarding_id` es el ID del primer *onboarding/intake form* que ya completó el paciente
- Lleva el tipo definido en la constante `Form::ONGOING_PROGRESS` que es igual a `ongoing`
- Tiene los mismo valores en `type_name` e `injury_name` del *onboarding/intake* inicial
## 2. Se guarda el Progress Form

Endpoint: `POST /forms/:uuid/ongoing_forms`

