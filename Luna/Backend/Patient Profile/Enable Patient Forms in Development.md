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

## ¿Qué hay que comentar?

En el modelo `Patients` de ActiveAdmin, hay que comentar la línea que hace que solo se pueda llamar al servicio de forms desde `Luna.env.live?`:

```ruby
begin
	forms_result = PatientFormsService.new.patient_onboarding_statuses([resource.id]) if Luna.env.live?
rescue PatientFormsServiceError => e
	ExceptionLogger.log(e, "Error fetching patient forms status: #{e}")
end
```

En la clase `PatientFormsService`, en el método `patient_self_report_service_enabled?` también hay que comentar la línea de `Luna.env.live?`:
```ruby
def patient_self_report_service_enabled?
  #return true if Luna.env.live?

  ENV["PATIENT_SELF_REPORT_SERVICE_DEVELOPMENT"] == "true"
end
```

Finalmente, en el archivo `.env` hay que setear en "true" la variable `PATIENT_SELF_REPORT_SERVICE_DEVELOPMENT`.

Con eso andando lo que queda es encontrar pacientes que tenga un Care Plan (Episode) reciente/activado.

# ¿Cómo encontrar pacientes con Care Plan reciente?

