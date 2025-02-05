# [EvW] Entidades para Pruebas

# Encontrar enlaces de Exercise Programs

Hay que buscar un `ExerciseProgram` que tenga varios registros de ejercicios.
```sql
select exercise_program_id, count(1) as cantidad
from exercise_assignments ea
group by exercise_program_id
having count(1) > 6;
```

Una vez se tenga un programa, se conserva el ID y se busca su `Episode` para obtener su `Patient`. Con ExerciseProgram y Patient se puede obtener el enlace usando el helper dispuesto para ello desde una consola de rails:

```ruby
ep = ExerciseProgram.find(UUID)
patient = ep.episode.patient

ExerciseProgramLinkHelper.generate_for(patient, ep)
```

# Para el endpoint GET exercise_programs

Es necesario el web_token que enlaza a la cuenta del `Patient` y el `exercise_program_id`:
```
  GET /api/v1/external/exercise_programs/:exercise_program_id?web_token=[WEB_TOKEN]
   
  GET /api/v1/external/exercise_programs/6e10e37c-1a08-4dfb-be44-f307d09c6c56?web_token=34997dfbasdiaser
```


# Para el endpoint `send_link_to_patient`

## Entidades para probar

```ruby
ExerciseProgram.find("6e10e37c-1a08-4dfb-be44-f307d09c6c56")

Patient.find("34997dfb-bec4-423c-8f94-cd0977b485ea").account.email
# Email: "francisco.quintero+25@ideaware.co"

Therapist.find("fffe332c-5c95-40df-b894-75813ffa7097").account.email
# Email: francisco.quintero+26@ideaware.co
```

**Detalles**

Este endpoint necesita autenticación de parte de un `Therapist` porque está en el namespace `v3/therapists` el cual es para al aplicación móvil.

Para la autenticación necesitamos encontrar el `Therapist` del `ExerciseProgram`.

Para hacerlo se usa el id almacenado en el atributo `assigned_by`.
```ruby
ExerciseProgram.find('6e10e37c-1a08-4dfb-be44-f307d09c6c56').assigned_by
=> "fffe332c-5c95-40df-b894-75813ffa7097"

Therapist.find("fffe332c-5c95-40df-b894-75813ffa7097")
```

Usando la `Account`  de ese therapist ya se pueden generar las credenciales para autenticar la petición:

```ruby
Therapist.find("fffe332c-5c95-40df-b894-75813ffa7097").account.create_new_auth_token

Account.find('fffe332c-5c95-40df-b894-75813ffa7097').create_new_auth_token
```

Para ejecutar la petición y generar el envío del enlace se necesitan:

- patient_id
- exercise_program_id

```bash
POST /api/v3/therapist/exercise_programs/6e10e37c-1a08-4dfb-be44-f307d09c6c56/send_link_to_patient?patient_id=ca107288-292e-4aed-b0db-734b69037ff7
    
ACCESS-TOKEN: t6mycS_eUSE-jyXSEV2e3w
TOKEN-TYPE: Bearer
CLIENT: AziFlODUXIcdO8WXLfrtgQ
EXPIRY: 1758851265
UID: francisco.quintero+26@ideaware.co
```

# Para prueba de envío de SMS

Lo más probable es que quede vacío el valor para `from` de los parámetros requeridos:

```ruby
TwilioHelper.send_sms(
	body: message,
	from: config["outbound_sms_number"][patient.region.name],
	to_contactable: patient.account
)
```

Cuando se prueba para el paciente:
```ruby
Patient.find("ca107288-292e-4aed-b0db-734b69037ff7")
```

Vemos que dicha configuración queda nula:
```ruby
config["outbound_sms_number"][patient.region.name]
nil
```

Al inspeccionar `config["outbound_sms_number"]` vemos esto en local

```json
{
	"los_angeles": "+15555555555",
	# (...)
	"fresno": "+15555555555"
}
```

Estos son los valores que obtengo en alpha:

```json
{"los_angeles"=>"+14242391511",
 "san_francisco"=>"+16504601220",
 "sdoc"=>"+16572109092",
 "seattle"=>"+12063851901",
 "chicago"=>"+17733056028",
 "detroit"=>"+19472220623",
 "phoenix"=>"+14806319276",
 "san_diego"=>"+16194042171"}
```

Y si vemos el valor para `patient.region.name`:
```ruby
patient.region.name
"out_of_area"
```

Podemos entender porque arroja nulo.

Para lograr avanzar en local, cambié la región del paciente en uso:
```ruby
Patient.find("ca107288-292e-4aed-b0db-734b69037ff7").update(region_id: 2)
```

Regiones
```
Los Angeles: 2
Fresno: 2981
Out of Area: 3
```

Y luego pude hacer la petición pero explota por falta de credenciales:
```
Error ExercisesViaWeb, SMS link: [HTTP 401] 20003 : Unable to create record
Authentication Error - No credentials provided
https://www.twilio.com/docs/errors/20003
```


# Usuario francisco.quintero+25@ideaware.co

Patient
```ruby
Patient.find("34997dfb-bec4-423c-8f94-cd0977b485ea")
```

Current CarePlan / Episode
```ruby
Patient.find("34997dfb-bec4-423c-8f94-cd0977b485ea").current_care_plan
```

Número de teléfono:
```ruby
# anterior:             4087094366
# nuevo (solo pruebas): 9149013263

Account.find_by(email: "francisco.quintero+25@ideaware.co").update_attribute(:phone_number, "9149013263")
```


# Usuario francisco.quintero+p@ideaware.co

Patient
```ruby
Account.find_by(email: "francisco.quintero+p@ideaware.co").patient
```

Current CarePlan / Episode
```ruby
Account.find_by(email: "francisco.quintero+p@ideaware.co").patient.current_care_plan
```

# Crear un Exercise Program

Para probar esta notificación hay que primero eliminar el ExerciseProgram existente. Una alternativa puede ser usar los métodos que invocan notificaciones directamente ya que son públicos.
```ruby
patient.current_care_plan.exercise_program.check_download_app_sms
patient.current_care_plan.exercise_program.check_download_app_email
```


## Condiciones a tener en cuenta para poder enviar el email al crear Exercise Program

```ruby
# tiene que estar en false
return if patient.download_app_exercise_email_sent.present?

# exercise_program checked_out en true
return unless checked_out_previously_changed? && checked_out?

# hay que modificar esto a cero
return if patient.account.sign_in_count.positive?
```

Cambiar el `sign_in_count` para el paciente
```ruby
Account.find_by(email: "francisco.quintero+25@ideaware.co").update_attribute(:sign_in_count, 0)
```


# Condiciones para los casos wilted tree y pending exercises

```ruby
# tiene que ser false para continuar
next if careplan.discharged?

# tiene que estar dentro de la region para continuar
next unless region_ids.include?(patient.region_id)

# no tiene que tener logins para continuar
# Solo para la Push
next if patient.account.sign_in_count.positive?

# deben ser iguales los Episodes/Care Plans
next unless careplan == patient.recent_care_plan

# debe tener workouts para continuar
next if workouts.empty?

# no debe tener workout reciente (menor a 8 días) para continuar
next if recent_workout_date.present?
```

Comprobando las condiciones con Pacientes
```ruby
account = Account.find_by(email: "francisco.quintero+p@ideaware.co")
patient = account.patient
care_plan = patient.recent_care_plan
ep = care_plan.exercise_program
workouts = ep.workouts.order(started_at: :desc)

# tiene que ser false para continuar
patient.episodes.last.discharged?

# tiene que estar dentro de la region para continuar
patient.region_id

# no tiene que tener logins para continuar
sign_in_count.positive?
# para cambiar count si es positivo
account.update_attribute(:sign_in_count, 0)

# deben ser iguales los Episodes/Care Plans
ep.episode.id == account.patient.recent_care_plan.id

# debe tener workouts para continuar
patient.episodes.last.exercise_program.workouts.count
# account.patient.recent_care_plan.exercise_program.workouts.count
# Para que tenga workouts hay que hacer una rutina en la página EvW

# NO debe tener workout reciente (menor a 8 días) para continuar
ep.workouts.collect(&:started_at).select { |date| date.present? && date > Time.current - 8.days }.first
# cambiando el start_date se puede solucionar para poder probar
```


## Sobre discharged?

Se define mediante los valores del enum `status`:
```ruby
# app/models/episode.rb

enum status: { active: 0, auto_discharged: 1, treatment_completed: 2, draft: 666 }
```

## Ejecutar el worker

Así se corre desde rails console:
```ruby
WiltedTreeExercisesViaWebReminderWorker.perform_async([1])
# region_id = 1
```

# Otros usuarios

## Paciente francisco.quintero+evw01@ideaware.co

```ruby
Account.find_by(email: "francisco.quintero+evw01@ideaware.co")
```

