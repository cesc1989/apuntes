# 024 - Forms para QA no aparecen en el perfil del paciente en Luxe

Etiquetas: #luna_help_desk 

Caso EDG-3112

Reporte: Ivan me contacta porque los forms en alpha demoran en salir cuando QA hace las pruebas. Demoran hasta horas en reflejarse.

## Contexto

Cuando se crea un nuevo Care Plan en Luxe se envía una petición a Marketplace para que se refleje y luego se cree el correspondiente Intake Form.

Ver [[Intake_Form_and_Progress_Form_creation_flow en Marketplace]]

Mediante una serie de peticiones el equipo de Automation procura la creación de los care plans. Esta petición es ejemplo:
```
POST https://luxe.alpha.getluna.com/automation/api/therapist/create_careplan
Body: {
  "patient_id": "b96c140c-bd60-4d94-a34b-88eb50102164",
  "careplan": {
    "injury_id": "11",
    "diagnosis_id": "4",
    "home_health": "Yes",
    "insurance_id": "57"
  },
  "first_name": "dorky fuchsia beagle"
}
```

### Mis Pruebas Manuales 🟢

Para comprobar hice algunas pruebas manuales.

Primero cree un nuevo paciente en Luxe y después completé un CCW en Bliss. Cuando volví a Luxe y recargué el link al intake form estaba presente.

También probé el endpoint usando Bruno. Busqué un paciente antiguo sin care plan. Mandé la 
petición, se creó el care plan y a los minutos apareció el form en el perfil del paciente.

O sea que los flujos normales siguen funcionando.

# Datos y Pacientes afectados en Alpha

Estos son los pacientes que reportó el tipo de QA:

```ruby
Patient.find("a917a5d7-9c61-4e8f-a28b-f94e83fa5aa2")

# Ya están bien.
Patient.find("0bbec751-8cb8-47f5-97c0-2d6b8962abf6")
Patient.find("62d7d014-1049-40de-ad13-bb76a589f482")
Patient.find("f6d70b5c-9ec9-4237-aa7a-114eec016d4b")
```

Cuando reviso si tienen más de un Intake Form veo que todos tienen solo uno:
```
irb(main):001> Patient.find("a917a5d7-9c61-4e8f-a28b-f94e83fa5aa2").episodes.count
=> 1
```

Al revisar si tienen el Intake Form en Marketplace aparece que varios ya tienen un Intake Form:
```
patient_id = 'a917a5d7-9c61-4e8f-a28b-f94e83fa5aa2'
Found 0 forms

patient_id = '0bbec751-8cb8-47f5-97c0-2d6b8962abf6'
Found 1 forms
Form: a176e191-35fd-4b86-b8ab-7624c2558f28, Type: intake

patient_id = '62d7d014-1049-40de-ad13-bb76a589f482'
Found 1 forms
Form: 1b50249c-e3f8-4b6c-bd15-04f9d5a88e70, Type: intake

patient_id = 'f6d70b5c-9ec9-4237-aa7a-114eec016d4b'
Found 1 forms
Form: 494e4d66-a174-4fac-94d2-7146ef56c6f6, Type: intake
```

Al revisar en Luxe, esos forms existen y se muestran en el perfil del paciente.

## Delay 🟡

Probé con un paciente antiguo que ya tenía un care plan discharged. Creé un nuevo care plan usando el endpoint de automation y se creó.

```ruby
Patient.find("9f7a773c-eed2-4a31-95ac-204934c02848").recent_care_plan
id: "28aa4c6a-1a32-4dcc-bf02-c37d766bfbab",
created_at: Wed, 10 Dec 2025 07:36:36.476683000 PST -08:00,
```

Pasaron varios minutos hasta que se creó el form:
```ruby
Patient.find("9f7a773c-eed2-4a31-95ac-204934c02848").recent_care_plan.onboarding_form

uuid: "1da0ff5f-ab28-4065-88bf-1d18ce4f9b2a",
created_at: Wed, 10 Dec 2025 07:49:56.967255000 PST -08:00,
care_plan_id: "28aa4c6a-1a32-4dcc-bf02-c37d766bfbab",
```

Hubo delay de 14 minutos aprox.