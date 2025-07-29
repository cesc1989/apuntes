# QA DART - Todas las Pruebas

Hay dos cosas a tener en cuenta:

- Escenarios de pruebas iniciales
- Escenarios de agendamiento (booking)

## Escenarios de Pruebas Generales

Lo que tiene que ver con crear un registro en `therapist_direct_access_entries` y que se pueda actualizar dicho registro.

- Nuevo Sign Up da error cuando el Therapist aún no existe
	- El Contacto se enrola en el workflow
	- Ingresa a la rama Active Attesting
	- Falla el webhook
	- Se completa el workflow
- Cuando se completa el Credentialing Form????
- Segundo Sign Up crea un nuevo registro
	- El Contacto se enrola en el workflow
	- Ingresa a la rama Active Attesting
	- Se completa el webhook
	- Se completa el workflow
- Actualizar propiedades selectas del Credentialing actualiza el registro
	- El Contacto se enrola en el workflow
	- Ingresa a la rama Active Attesting
	- Se completa el webhook
	- Se completa el workflow