# EDG-3279 Multiples Intake Forms creados para un mismo Care Plan

Etiquetas: #luna_help_desk 

## Reporte

Reportaron que los pacientes completan Intake Forms pero siguen saliendo otros pendientes por completar. El error aquí es que se están creando varios Intake Forms para un mismo Care Plan.

# Problema

El problema es que por alguna razón Marketplace estaba enviando varias peticiones para crear el Intake Form. Esto estaba creando varias veces Intake Forms para un mismo Care Plan. Algo que es inválido en el sistema.

## Datos


## Contexto

Marketplace puede enviar peticiones a los dos endpoints disponibles para crear Intake Forms.

Los endpoints son:
- 1er Intake Form: `{{BASE_URL}}/patient_forms`
- 2do Intake Form: `{{BASE_URL}}/patients/:patient_internal_id/request_intake_forms`

La lógica está en `marketplace/forms/controller.py`:
```python
def create_intake_form(
        self, now: pendulum.DateTime, patient_result: es_repository.PatientResult, care_plan_result: es_repository.CarePlanResult
    ) -> models.Form:
        """Create an intake form for a patient and care plan."""
        existing_forms = list(self._repository.get_all_patient_forms(patient_result.aggregate_id))

        if not existing_forms:
            intake_form_result = self._service.create_intake_form_new_patient(patient_result.patient, care_plan_result.care_plan)
        else:
            intake_form_result = self._service.create_intake_form_existing_patient(patient_result.patient, care_plan_result.care_plan)

        intake_status_result = self._service.get_form_status(patient_result.patient, intake_form_result.status_id)
        return self._repository.create_form(now, care_plan_result.aggregate_id, intake_form_result, intake_status_result)
```

En la función `create_intake_form_existing_patient` es donde se hace la petición.

# Soluciones

La solución está por agregar una validación en el endpoint del 2do Intake Form para prevenir que se cree si ya existe uno para el `care_plan_id`.

## Sobre el campo `exclude_medical_information`

Creí que habría problema por esto pero corrí una query para ver cuántos Intake Forms tienen este campo en true y resulta que no hay uno solo.

```sql
select count(f.id)
from forms f
where f.exclude_medical_information is true;
```

**O sea que esto no está en uso. Nunca se ha usado.**