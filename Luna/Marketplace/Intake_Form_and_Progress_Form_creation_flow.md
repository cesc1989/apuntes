# Intake Form and Progress Form creation flow

Other References:
- [[Navegando_creación_de_intake_form]]

# Intake Form

After a new Care Plan is created, a job to create a new intake form is triggered.

This can be seen in [app/marketplace/forms/listeners.py](https://github.com/lunacare/marketplace/blob/omega/app/marketplace/forms/listeners.py#L21-L32)

```python
    def create_intake_form(sender: Any, session: session.Session, aggregate_id: str, **extra: dict) -> None:
        """Creates an intake form for a patient when a new case is created."""
        logging.logger().info(f"create_intake_form → signal received: /case_created/ for aggregate {aggregate_id}")
    
        job = _queue().enqueue(
            jobs.create_intake_form,
            kwargs={"aggregate_id": aggregate_id},
            job_timeout=_JOB_TIMEOUT_FORMS_OPERATION,
            retry=rq.Retry(max=3, interval=[10, 30, 60]),
        )
    
        utility.log_job_enqueued(logging.logger(), "forms_listeners", job, aggregate_id=aggregate_id)
```

The trigger is the signal `CASE_CREATED`.

Signal definition: https://github.com/lunacare/marketplace/blob/omega/app/marketplace/signals.py#L24-L25

```python
    # New care plan has been created for a patient.
    CASE_CREATED = _signal_namespace.signal("case-created")
```

Care Plan Created event definition: https://github.com/lunacare/marketplace/blob/omega/app/marketplace/es/events/internal.py#L97-L109

```python
   @attr.s(frozen=True, slots=True)
    class CarePlanCreated:
        EVENT_TYPE = "care_plan_created"
    
        care_plan_id = attr.ib()
        patient_id = attr.ib()
        affected_body_part = attr.ib()
        care_plan_number = attr.ib()
        surgery_date = attr.ib(converter=utility.ensure_pendulum_date)
        clinic_id = attr.ib(converter=attr.converters.optional(str))
    
        @property
        def signals(self) -> List[blinker.Signal]:
            return tuple([signals.CASE_CREATED])
```

Care Plan Created event: https://github.com/lunacare/marketplace/blob/omega/app/marketplace/es/aggregates.py#L191-L209

```python
   @attr.s(auto_attribs=True)
    class CarePlan:
        _events: List[Any]
        changes: List[Any] = attr.Factory(list)
    
        def __attrs_post_init__(self):
            for event in self._events:
                self._apply(event)
    
        @classmethod
        def create(
            cls, care_plan_id: str, patient_id: str, affected_body_part: str, care_plan_number: int, surgery_date: pendulum.Date, clinic_id: int
        ) -> "CarePlan":
            create_initial_care_plan = events.CarePlanCreated(
                care_plan_id, patient_id, affected_body_part, care_plan_number, surgery_date, clinic_id
            )
            instance = cls([create_initial_care_plan])
            instance.changes = [create_initial_care_plan]
            return instance
```

Blueprint where care plan creation starts: https://github.com/lunacare/marketplace/blob/omega/app/marketplace/es/blueprint.py#L294-L340

```python
  def upsert_care_plan(patient_id, care_plan_id, upsert_request, enqueue_event=True):
        # (...)
    
        change_context.process_signals(enqueue_event=enqueue_event)
    
        visit_plan.invalidate_cached_visit_plan(updated_care_plan.care_plan_id)
    
        return {"id": care_plan_id, "aggregate_id": change_context.aggregate_id}
```

The `upsert_care_plan` is invoked in a function in the same file: https://github.com/lunacare/marketplace/blob/omega/app/marketplace/es/blueprint.py#L274-L291

```python
   @sync_api.route("/api/v1/internal/sync/patients/<patient_id>/care-plans/<care_plan_id>", methods=["PUT"])
    def sync_care_plan(patient_id, care_plan_id):
        """Syncs a care plan and any missing details.
    
        Clients are expected to call this endpoint any time an appointment action is performed (new, updated, or cancelled).
        """
        json_data = flask.request.get_json(force=True)
    
        upsert_schema = schemas.UpsertCarePlanRequestSchema()
        upsert_request = upsert_schema.load(json_data)
    
        utility.log_debug_data(logging.logger(), "sync.views.sync_care_plan", "request", json_data)
        with (
            utility.suppress_lock_errors(),
            utility.ensure_single_execution(f"sync_patient({patient_id})"),
        ):
            return upsert_care_plan(patient_id, care_plan_id, upsert_request, enqueue_event=json_data.get("enqueue_event", True))
        return {}
```

And here it’s where it seems to start. Backend call to Marketplace: https://github.com/lunacare/backend/blob/omega/app/services/marketplace.rb#L553-L564

```python
  def sync_care_plan(care_plan_id)
        return OpenStruct.new(success?: true) unless marketplace_enabled?
    
        care_plan = Episode.find(care_plan_id)
    
        result = self.class.put("/api/v1/internal/sync/patients/#{care_plan.patient_id}/care-plans/#{care_plan.id}", {
          body: emr_care_plan_params(care_plan)
        })
        raise MarketPlaceException, result.inspect unless result.success?
    
        result
    end
```


# Progress Form

A new progress form after a therapist visit finishes.

This can be seen in https://github.com/lunacare/marketplace/blob/omega/app/marketplace/forms/listeners.py#L35-L52

```python
   def create_progress_form(
        sender: Any, session: session.Session, aggregate_id: str, event: events.TherapistStartedSession, **extra: dict
    ) -> None:
        """Creates a progress form for a patient when a visit is finished."""
        logging.logger().info(
            f"create_progress_form → signal received: /visit_finished/ for appointment {event.appointment_id} / aggregate {aggregate_id}"
        )
    
        job = _queue().enqueue(
            jobs.create_progress_form,
            kwargs={"aggregate_id": aggregate_id, "appointment_id": event.appointment_id},
            job_timeout=_JOB_TIMEOUT_FORMS_OPERATION,
            retry=rq.Retry(max=3, interval=[10, 30, 60]),
        )
    
        utility.log_job_enqueued(
            logging.logger(), "forms_listeners", job, aggregate_id=aggregate_id, appointment_id=event.appointment_id, final="false"
        )
```

The trigger in this case is signal `VISIT_FINISHED`.

The event definition https://github.com/lunacare/marketplace/blob/omega/app/marketplace/es/events/internal.py#L315-L323

```python
   @attr.s(frozen=True, slots=True)
    class TherapistFinishedSession:
        EVENT_TYPE = "therapist_finished_session"
    
        appointment_id = attr.ib()
    
        @property
        def signals(self) -> List[blinker.Signal]:
            return tuple([signals.VISIT_FINISHED])
```

Here is the code that triggers the event https://github.com/lunacare/marketplace/blob/omega/app/marketplace/es/aggregates.py#L325-L328

```python
  @_apply.register(events.TherapistStartedSession)
    def _change_appointment_status_in_progress(self, event: events.TherapistStartedSession) -> None:
            i, appointment = self._appointment_index(event.appointment_id)
            self.appointments[i] = attr.evolve(appointment, status=AppointmentStatus.THERAPIST_STARTED)
    
        @_apply.register(events.TherapistFinishedSession)
```
