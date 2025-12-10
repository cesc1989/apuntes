# Cómo habilitar Patient Forms en Development

> [!Important]
> Ya que se completó el AppBlend de Patient Self Report en Edge esto ya no aplica.
> El servicio `PatientFormsService` no existe más. Ni tampoco verificación de si `Luna.env.live?` para poder acceder a los Forms.

Para poder probar la funcionalidad de Patient Forms integrada con Luxe toca primero activar una ENV y comentar algunas líneas de código.

## Archivos Involucrados

Estos son los archivos donde está todo lo relacionado con el enlace entre Patient Forms y Luxe

- Modelo ActiveAdmin `admin/customers/patients.rb`
	- sección `show do`
	- sección `panel "Details"`
- Partial `views/admin/patients/_info.html.erb`
	- Buscar "Latest Forms"
- Clase `PatientFormsService`
	- Ir al fondo, método `patient_self_report_service_enabled?`

Ejemplo de perfil del paciente

![[01.patient.profile.forms.png]]

- El recuadro rojo es lo que se configura en `panel "Details"`.
- Si en la sección "Care Plan" aparece el botón _Edit Care Plan_ es que tiene Care Plan reciente.
