# REM Accurate Returning Patients

Autor: Alexis Ramis

## Contexto

El widget de Unseen Patients en Clinical Dashboard le faltan datos. Ahora se quiere más precisión para mostrar pacientes que no han sido agendados. También para atender a pacientes que volvieron.

### Problemas

- Usa la propiedad `hs_lead_status` la cual no es del todo fiable
- Usa la propiedad `createdate` para el filtro pero no es del todo correcta
- Hay constantes en muchos lugares. Tal vez desincronizadas.

El campo correcto para entender el estado es `last_case_booking_outcome`.

## Objetivos

- Cambiar peticiones API directas y usar mejor `Hubspot::Connection`
- Cachear respuestas de API de HS
- Usar nuevas propiedades
- Mejorar los permisos de acceso para mayor seguridad

## Propuesta

### Mejora de Authorization

Configurar `current_clinical_dashboard_user` para determinar el acceso a registros y a hubspot_ids.

### Estandarizar API Calls

Usar `Hubspot::Connection` en vez de peticiones API normales.

### Actualizar propiedad de estado

Usar `last_case_booking_outcome` en vez `hs_lead_status`.

### Nuevo campo de referencia de fecha

Crear nueva propiedad en HubSpot para tener una mejor referencia de la fecha para los filtros. Llevará backfill y workflow para su llenado.

El campo será `date_last_case_booking_outcome_set`.

## Milestones

### M0 - Authorization

- Configurar Active Admin para crear usuarios de Clinical Dashboard Admin
- Configurar `current_clinical_dashboard_user` para acceder mediante este a los registros correspondientes
	- Podría necesitar establecer una asociación entre ClinicalDashboard::User y Physician/Practice/Clinic

> [!Warning]
> Eso último es de comentar porque esa relación no es del todo necesaria.


### M1 - Cleanup

- Usa la gema Hubspot para las peticiones
- Organiza las constantes
- Verifica las propiedades están al día con la rake de Hubspot
- Configura cacheo de respuesta de peticiones

### M2 - Fixes

- Mejoras de los filtros de 90 días y all time.\

### M3 - Nuevo HS Field, Workflow y Backfill

- Todo con respecto a agregar el nuevo campo de fecha `date_last_case_booking_outcome_set`.
- Workflow que la actualice
- Backfill para contactos existentes

### M4 - Usar las propiedades creadas/actualizadas

- Replace `hs_lead_status` with `last_case_booking_outcome` for booking status filtering
- Replace `createdate` with `date_last_case_booking_outcome_set` for 90-day filtering