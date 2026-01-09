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

Otro caso de Form con delay:
```ruby
# Luxe
care plan created at: Wed, 10 Dec 2025 14:15:55.715677000 PST -08:00

ID: fa47bcfc-dc5f-4aa5-b4de-078bb3b8f069
created_at: 2025-12-10 14:17:57 -0800
care_plan_id: afe353a0-2118-4b08-b608-49506d8307e9

# Marketplace
Found 1 forms
Form: fa47bcfc-dc5f-4aa5-b4de-078bb3b8f069, Type: intake, CreatedAt: 2025-12-10 22:17:57.367877+00:00
```

En este caso el delay fue del job saliendo de backend hacía marketplace.

## Prueba Directa

Decidí probar la petición directamente sin pasar por el worker y el sync es inmediato.

Ejemplo 1:
```ruby
# Luxe
care_plan: 2025-12-10 17:26:28 -0800

# Form
ID: a4161ee5-802b-4d66-b12e-8b517d7ee9cc
created_at: 2025-12-10 17:26:29 -0800
care_plan_id: 5897bea3-591e-4933-aacf-6562739f6a1c

# Marketplace
Found 1 forms
Form: a4161ee5-802b-4d66-b12e-8b517d7ee9cc,
Type: intake,
CreatedAt: 2025-12-11 01:26:28.937904+00:00
```

> [!Info]
> Actualización, 12/12/25: hice varias pruebas y directa y tampoco mejoró la cosa. Hubo demora aunque en una ocasión sí se creó el form.



## Mejora Momentanea

Hoy, 10 de Diciembre, pedí a QA que probara y los forms se generaron casi que instantáneamente.

Ejemplo 1:
```ruby
# Luxe
ID: 1d85f3fc-dddf-4fcf-bfc0-4d5d97cf6139
created_at: 2025-12-10 11:53:52 -0800
care_plan_id: ee909b71-e5ca-4e80-9eac-1a20b55f670f

# Marketplace
Found 1 forms
Form: 1d85f3fc-dddf-4fcf-bfc0-4d5d97cf6139
Type: intake,
CreatedAt: 2025-12-10 19:53:52.584974+00:00
```

Ejemplo 2:
```ruby
# Luxe
ID: 1f93bd03-e567-4541-af95-76dde5bb202b
created_at: 2025-12-10 12:12:53 -0800
care_plan_id: 4723eb22-5f5b-4f48-b1b0-de2e6e9f4a09

# Marketplace
Found 1 forms
Form: 1f93bd03-e567-4541-af95-76dde5bb202b
Type: intake
CreatedAt: 2025-12-10 20:12:53.362679+00:00
```

# Situación con Sidekiq 🚧🚧

Definitivamente pasa algo aquí con Sidekiq.

- En ocasiones Sidekiq tardaba varios segundos en tomar el worker
- Veo que la cantidad de procesos puede llegar hasta los 23 y bajar hasta los 3

Ver [[Comandos Sidekiq]] para saber como debuggear en Alpha desde una consola de Rails.

## Demora en tomar los jobs

Para un caso que probé en la mañana, al revisar la cola de Sidekiq para el worker `MarketplaceSyncCarePlanWorker` veía esto:

```
Care Plan ID: 95114e0c-80e9-488f-b20f-03600226ff25, Enqueued at: 2025-12-12 15:27:39 UTC
Care Plan ID: 13e8e3fb-753d-49dd-8188-cd40c98bcbd3, Enqueued at: 2025-12-12 15:23:24 UTC
Care Plan ID: a53d40ea-26db-4740-b1c1-2ee372d0088f, Enqueued at: 2025-12-12 15:23:24 UTC
```

Donde me interesaba que el care plan id terminación `03600226ff25` fuera ejecutado. Pero al probar de nuevo el script veía que seguía encolado:
```
Care Plan ID: a6f51428-16a9-4489-bdf6-01cd1696b838, Enqueued at: 2025-12-12 15:35:48 UTC
Care Plan ID: 95114e0c-80e9-488f-b20f-03600226ff25, Enqueued at: 2025-12-12 15:27:39 UTC
Care Plan ID: 13e8e3fb-753d-49dd-8188-cd40c98bcbd3, Enqueued at: 2025-12-12 15:23:24 UTC
Care Plan ID: a53d40ea-26db-4740-b1c1-2ee372d0088f, Enqueued at: 2025-12-12 15:23:24 UTC
```

El resultado al final cuando al fin se ejecutó. Resumen de Claudio.

Timing Analysis for Care Plan: 95114e0c-80e9-488f-b20f-03600226ff25

**ENQUEUE**: 2025-12-12T15:27:39.316Z
**START**: 2025-12-12T15:42:04.110Z
**DONE**: 2025-12-12T15:43:56.096Z

Breakdown:

| Phase                      | Duration     | Notes                                           |
|----------------------------|--------------|-------------------------------------------------|
| Sidekiq Queue Wait         | 14.4 minutes | Job sat in queue waiting for worker             |
| Marketplace Sync Execution | 1.9 minutes  | Actual work (syncing to Marketplace)            |
| Total Backend Time         | 16.3 minutes | Before Marketplace can even start form creation |