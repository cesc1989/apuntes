# Navegando creación de intake_form

No me queda claro si todo se crea a partir de la tarea programada en `rq_scheduler.py`
```python
# Create missing intake forms
scheduler.cron(_CRON_SCHEDULE_EVERY_30_MINUTES, func=forms_tasks.create_missing_intake_forms, use_local_timezone=True)
```

De ahí pasamos a `marketplace/forms/tasks.py`
```python
def create_missing_intake_forms() -> None:
	"""Creates forms that were not created for some unknown reason (data corruption)."""
	with session_scope() as session:
			aggregates_missing_intakes_query = (
					session.query(es_models.Aggregate.id)
					.join(es_models.IntegratedCarePlan)
					.join(forms_models.Form, isouter=True)
					.filter(forms_models.Form.id.is_(None))
			)
			aggregate_ids = [str(r[0]) for r in aggregates_missing_intakes_query]

	for aggregate_id in aggregate_ids:
			forms_jobs.create_intake_form(aggregate_id)
			time.sleep(2.5)
```

Sigue el trabajo en segundo plano donde se buscan el care plan y el patient. Esto pasa en `marketplace/forms/jobs.py`
```python
def create_intake_form(aggregate_id: str) -> None:
		with database.session_scope() as session:
				es_repo = es_repository.Repository(session)

				care_plan_result = es_repo.get_care_plan_aggregate(aggregate_id)
				patient_result = es_repo.get_patient_aggregate_from_external_id(care_plan_result.care_plan.patient_id)

				forms_repository = repository.Repository(session)
				forms_controller = controller.Controller(forms_repository, patient_form)
				forms_controller.create_intake_form(pendulum.now(), patient_result, care_plan_result)
```

Aquí se hace la petición al endpoint en el controlador en `marketplace/forms/controller.py`
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


Finalmente, llegamos a `marketplace/services/patient_form.py`. En esta función se reciben las instancias, se arma el JSON y se hace la petición a **patient-self-report**

```python
def create_intake_form_new_patient(patient: aggregates.Patient, care_plan: aggregates.CarePlan) -> entities.FormResponse:
		"""Create an intake form for a new patient."""
		endpoint = f"{settings.FORMS.API_BASE_URI}patient_forms/"
		json_data = {
				"patient": {
						"first_name": patient.first_name,
						"last_name": patient.last_name,
						"internal_id": patient.patient_id,
						"date_of_birth": str(patient.date_of_birth),
						"gender": patient.gender,
						"email": patient.email_address,
						"forms_attributes": [
								{
										"type_name": MedicalFormType.from_body_part(care_plan.affected_body_part),
										"injury_name": care_plan.common_label,
										"care_plan_id": str(care_plan.care_plan_id),
								}
						],
				}
		}

		log_request_debug("create_intake_form_new_patient", endpoint, json_data)

		with Session() as session:
				create_form_response = session.post(endpoint, json=json_data)
				if not create_form_response.ok:
						raise PatientFormsUpstreamError(f"failed to create intake form for new patient: {create_form_response.reason}")

				response_json = create_form_response.json()

		log_response_debug("create_intake_form_new_patient", endpoint, response_json)

		onboarding_response_schema = schemas.CreateFormResponseSchema()
		onboarding_response_schema.context = {"form_type": "intake"}
		onboarding_response = onboarding_response_schema.load(response_json)

		return onboarding_response
```

