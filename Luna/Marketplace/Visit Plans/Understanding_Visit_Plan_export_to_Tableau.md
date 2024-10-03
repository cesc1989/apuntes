# Understanding Visit Plan export to Tableau

The deal is in:
```python
   # app/marketplace/visit_plan/jobs.py
    
    def write_visit_plans_to_data_lake() -> None:
```

In this document, I’ll put comments about specific parts, how they work, learnings, and final understanding of this Python system.

# Querying Visit Plans

Let’s try to understand this code:
```python
    visit_plans_query = (
      session.query(models.VisitPlan)
      .filter(~models.VisitPlan.published_at.is_(None))
      .filter(models.VisitPlan.appointment_date >= pendulum.DateTime(2022, 1, 1).in_timezone("UTC"))
    )
```

The line with `session.query(models.VisitPlan)` sets the VisitPlan model to be queried against.

In the next line:
```python
.filter(~models.VisitPlan.published_at.is_(None))
```

what’s inside the filter is a [NOT IN statement](https://docs.sqlalchemy.org/en/14/core/operators.html#not-in). It’s fetching VisitPlan that aren’t published.

Next line:
```python
.filter(models.VisitPlan.appointment_date >= pendulum.DateTime(2022, 1, 1).in_timezone("UTC"))
```

continues the filters to return only VisitPlan whose appointment_date is greater than date January 1st, 2022.

This date is created with Pendulum library.

Notice the whole expression is in multiple lines wrapped by parentheses. This is a way to achieve line continuation in Python. [Explained by ChatGPT.](https://chat.openai.com/share/ee750c34-0408-4ca8-a5b7-a95b8c32c3c3)

Finally, `**visit_plans_query**` is a query object that looks to only hold the SQL query but not records:
```python
  (Pdb) visit_plans_query
    <sqlalchemy.orm.query.Query object at 0x146005fa0>
    
    (Pdb) print(visit_plans_query)
    SELECT marketplace.visit_plan.id AS marketplace_visit_plan_id, marketplace.visit_plan.created_at AS marketplace_visit_plan_created_at, marketplace.visit_plan.published_at AS marketplace_visit_plan_published_at, marketplace.visit_plan.continuation AS marketplace_visit_plan_continuation, marketplace.visit_plan.care_plan_aggregate_id AS marketplace_visit_plan_care_plan_aggregate_id, marketplace.visit_plan.anticipated_discharge_date AS marketplace_visit_plan_anticipated_discharge_date, marketplace.visit_plan.plan_text AS marketplace_visit_plan_plan_text, marketplace.visit_plan.prescribed_by_id AS marketplace_visit_plan_prescribed_by_id, marketplace.visit_plan.appointment_id AS marketplace_visit_plan_appointment_id, marketplace.visit_plan.appointment_date AS marketplace_visit_plan_appointment_date 
    FROM marketplace.visit_plan 
    WHERE marketplace.visit_plan.published_at IS NOT NULL AND marketplace.visit_plan.appointment_date >= %(appointment_date_1)s
```


# Visit Plan in Chunks

**NOTICE:** to run the visit_plan job in flask shell.

```python
from marketplace.visit_plan import jobs;jobs.write_visit_plans_to_data_lake()
```

Follow these lines:
```python
    for visit_plan_chunk in utility.chunks(
        utility.windowed_query(visit_plans_query, models.VisitPlan.created_at, window_size=25_000), size=25_000
    ):
        visit_plans = list(visit_plan_chunk)
        care_plan_aggregate_ids = [str(plan.care_plan_aggregate_id) for plan in visit_plans]
        care_plan_results = es_repo.get_care_plan_aggregates(care_plan_aggregate_ids)
        care_plan_results_map = {str(care_plan_result.aggregate_id): care_plan_result for care_plan_result in care_plan_results}
```

## First: utility.windowed_query

Let’s see first the query `windowed_query` part:
```python
utility.windowed_query(visit_plans_query, models.VisitPlan.created_at, window_size=25_000)
```

The docs of this method says:

> Breaks a query into chunks on a given column

When I try to print the windowed query get this:
```python
   utility.windowed_query(visit_plans_query, models.VisitPlan.created_at, window_size=25_000)
    <generator object windowed_query at 0x14401b900>
```

This is converted to an iterator in `utility.chunks` and then looped in batches of 25_000 records:
```python
for visit_plan_chunk in utility.chunks(windowed_query, size=25_000)
```

So this is like some kind of iterator inside another iterator. I can’t seem to find a way to inspect what’s actually being returned.


## Second: visit_plans list

So far what I get is that SQL Alchemy does not return objects as ActiveRecord but some kind of reference(?). This is what I see when printing a plan in `visit_plans` list:
```python
   visit_plans = list(visit_plan_chunk)
    
    for plan in visit_plans:
      logging.logger().info("IMPRIMIENDO LISTA VISIT PLANS")
      print(plan)
      logging.logger().info("TERMINO IMPRIMIENDO LISTA VISIT PLANS")
    
    INFO:marketplace.logging:IMPRIMIENDO LISTA VISIT PLANS
    <marketplace.visit_plan.models.VisitPlan object at 0x13fd81e50>
    <no request_id> : TERMINO IMPRIMIENDO LISTA VISIT PLANS
```

This is confirmed in next lines where attributes in `visit_plans` can be accessed by looping through it.

## Third: care_plans and care_plan_aggregates

When printing this in the shell:
```python
   care_plan_aggregate_ids = [str(plan.care_plan_aggregate_id) for plan in visit_plans]
```

I can see individual IDs being returned to fill in the array.

```python
   (Pdb) care_plan_aggregate_ids
    ['57e6a468-0a15-405f-9e75-85ba72d1f65b', 'd2d07b6d-650e-46b6-abff-b67dcc969735',...]
```

These IDs are then processed by `es_repo.get_care_plan_aggregates` where a lot of stuff happen.

```python
care_plan_results = es_repo.get_care_plan_aggregates(care_plan_aggregate_ids)
```

This process returns instances of custom class `CarePlanResult`:
```python
class CarePlanResult:
		aggregate_id: uuid.UUID
		version: int
		created_at: pendulum.DateTime
		care_plan: aggregates.CarePlan
```

Notice the last attribute in `CarePlanResult` correspond to instance of the `CarePan` aggregate class:
```python
class CarePlan:
		_events: List[Any]
		changes: List[Any] = attr.Factory(list)
```

This an example:
```python
[
	CarePlanResult(
		aggregate_id=UUID('72610e85-8ede-4e97-a286-bef5419da08d'),
		version=5,
		created_at=datetime.datetime(2022, 1, 14, 23, 59, 42, 291730),
		care_plan=CarePlan(
			_events=[
				CarePlanCreated(
					care_plan_id='0eabca0a-7d62-474a-9795-be53a6126f7c',
					patient_id='1687e74a-bb01-4ba9-9d2c-b2a14427afaf',
					affected_body_part='knee_joint_replacement',
					care_plan_number=1,
					surgery_date=Date(2022, 1, 5),
					clinic_id='-1'
				),
				PhysicianAttached(
					physician_id='7e6fe728-a2f8-4e75-9935-cf2384999385',
					npi='1598711681',
					first_name='Aaron',
					last_name='Salyapongse'
				),
				ClinicChanged(clinic_id='2'),
				AppointmentAdded(
					appointment_id='7cf24736-eb0e-4982-8ff4-ef2da645bbff',
					scheduled_time=DateTime(2022, 1, 13, 7, 15, 0, tzinfo=Timezone('-08:00')),
					duration_minutes=60,
					clinicient_staff_id='29',
					type='initial',
					status='scheduled'
				),
				TherapistStartedSession(appointment_id='7cf24736-eb0e-4982-8ff4-ef2da645bbff'),
				TherapistFinishedSession(appointment_id='7cf24736-eb0e-4982-8ff4-ef2da645bbff')
			],
			changes=[]
		)
	)
]
```

## Fourth: Care plan aggregate ID mapping to CarePlanResult

Final step, before going into the process that calls the section that times out it his snippet:
```python
    care_plan_results_map = {str(care_plan_result.aggregate_id): care_plan_result for care_plan_result in care_plan_results}
```

Here we get a dictionary where every ID in `care_plan_aggregate_ids` is mapped to its corresponding instance of `CarePlanResult`.

# Plan to Entity: the bottleneck

In this step, it takes each `CarePlanResult` and process them into a `VisitPlan.to_entity` value.

```python
return entities.VisitPlan(
		published_at=published_at,
		visit_plan_id=self.id,
		patient_id=care_plan_result.care_plan.patient_id,
		care_plan_id=care_plan_result.care_plan.care_plan_id,
		appointment_id=self.appointment_id,
		prescribed_by_id=self.prescribed_by_id,
		plan_text=self.plan_text,
		anticipated_discharge_date=anticipated_discharge_date,
		entries=entries,
		continuation=self.continuation,
		visits_remaining=visits_remaining,
		total_expected_visits=total_expected_visits,
)
```

This `VisitPlan` entity is defined in `app/marketplace/visit_plan/entities.py`:
```python
@attr.s(frozen=True, auto_attribs=True, kw_only=True)
class VisitPlan:
		"""Container of visit plan entries."""

		published_at: Optional[pendulum.Date]
		visit_plan_id: str
		patient_id: str
		care_plan_id: str
		appointment_id: str
		prescribed_by_id: str
		plan_text: str
		entries: List[VisitPlanEntry]
		anticipated_discharge_date: pendulum.Date
		continuation: bool
		visits_remaining: Optional[int]
		total_expected_visits: Optional[int]
```

# Use Matrix Model instead of ES Repository CarePlanResult

Ryan said:

> I think you need to do a refactor so that models/visit_plan to_entity takes in matrix_models.Appointment instead of es_repository.CarePlanResult

I need to understand how are SQL Alchemy models built as I don’t see full attributes definition on both of the mentioned models.

## Matrix Model Appointment

In `app/marketplace/matrix/models.py`, line 611 I see the model definition and some columns definitions.

```python
class Appointment(Base):
		__tablename__ = "appointment"

	id
	therapist_id
	care_plan_id
	appointment_id
	# inherited
	destination_id
	created_at
	time_start
	time_end
	therapist
	care_plan
	destination
```

However how do I know these match what’s actually in used for the current export?

# Current Export Flow

![[visitplan.flow.2.png]]
