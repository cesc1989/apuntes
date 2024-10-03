# Visit Plans data to Athena

This is setup to be exported and can be seen at several files.

Scheduled to run
```python
# app/marketplace/rq_scheduler.py

from marketplace.visit_plan import tasks as visit_plan_tasks

# Write visit plan data to S3
scheduler.cron(_CRON_SCHEDULE_DAILY_5_AM, func=visit_plan_tasks.write_visit_plans_to_data_lake, use_local_timezone=True)
```

The task
```python
# app/marketplace/visit_plan/tasks.py

q = rq.Queue("default", connection=cache._redis_jobs_client())
job = q.enqueue(jobs.write_visit_plans_to_data_lake, job_timeout=7200)
utility.log_job_enqueued(logging.logger(), "visit_plan_tasks", job)
```

The job
```python
# app/marketplace/visit_plan/job.py
    
# (...)

s3_object_path = f"{settings.AWS.DATA_LAKE_BUCKET}/visit-plans/latest/data.csv"

# (...)
```

But nothing appears in Sentry (in case of errors), the folder `visit-plans` in the DATA LAKE bucket does not exist. In Grafana, the logs does not show the expected logged string:
```python
logging.logger().info(f"wrote {bytes_written} bytes of visit plan data to {s3_object_path}")
```

# Inspecting DATA_LAKE env var

*UPDATE: fixed on October 2nd.* [*Pull Request*](https://github.com/lunacare/marketplace/pull/4604)*.*

Launch a Flask shell we need to run this command:
```python
FLASK_APP=marketplace/application.py flask shell
```

To verify the env var is setup we have to use the library `os`. The `DATA_LAKE` env is supposed to be set in `settings.AWS.DATA_LAKE_BUCKET`:
```python
# app/marketplace/config/base.py

@attr.s(slots=True, auto_attribs=True)
class AWSSettings:
		OCR_PROCESSING_S3_BUCKET: str = os.environ.get("OCR_PROCESSING_S3_BUCKET")
		DATA_LAKE_BUCKET: str = os.environ.get("DATA_LAKE_S3_BUCKET")
		REGION: str = os.environ.get("AWS_REGION", os.environ.get("AWS_DEFAULT_REGION"))
```

When I run the instruction to fetch the ENV var `**DATA_LAKE_S3_BUCKET**` I get nothing
```python
import os
os.environ.get("DATA_LAKE_S3_BUCKET")
```

The env var is not set and the job never completes.

Links:

- [How to check env vars in Python](https://stackoverflow.com/questions/4906977/how-can-i-access-environment-variables-in-python)

# Exploring Grafana logs

Using `ecs_clusters`
```
{ecs_cluster="marketplace-workers"} |= `visit plan`

{ecs_cluster="marketplace-workers"} |= `visit_plan_tasks`

# Nombre de la tarea
{ecs_cluster="marketplace-workers"} |= `write_visit_plans_to_data_lake`
```

Using `job`

    {job="alpha-marketplace"} |= `visit plan`
    
    {job="alpha-marketplace"} |= `visit_plan_tasks`
    
    # Nombre de la tarea
    {job="alpha-marketplace"} |= `write_visit_plans_to_data_lake`

Notice:

- Se puede buscar logs por el nombre de la tarea
- Sí aparecen logs para la de visit plan
- Se pueden hacer dos queries en Grafana

# Running job directly in Flask shell

*UPDATE: fixed on October 23rd.* [*Pull request*](https://github.com/lunacare/marketplace/pull/4633)*.*

Try to run the function directly from the shell and get this:
```python
from marketplace.visit_plan import jobs
jobs.write_visit_plans_to_data_lake()

Traceback (most recent call last):
	File "/usr/local/lib/python3.10/code.py", line 90, in runcode
		exec(code, self.locals)
	File "<console>", line 1, in <module>
	File "/app/marketplace/visit_plan/jobs.py", line 36, in write_visit_plans_to_data_lake
		with smart_open.open(s3_object_path, "wb") as file_out:
	File "/usr/local/lib/python3.10/site-packages/smart_open/smart_open_lib.py", line 177, in open
		fobj = _shortcut_open(
	File "/usr/local/lib/python3.10/site-packages/smart_open/smart_open_lib.py", line 363, in _shortcut_open
		return _builtin_open(local_path, mode, buffering=buffering, **open_kwargs)
FileNotFoundError: [Errno 2] No such file or directory: 'luna-alpha-workloads-data-lake/business-operations/visit-plans/latest/data.csv'
```

It’s an error using the library [smart-open](https://pypi.org/project/smart-open/).


# Enqueue the worker in Flask shell

Doing this
```python
import rq
from marketplace import database, logging, settings, utility
from marketplace.services import cache

q = rq.Queue("default", connection=cache._redis_jobs_client())
job = q.enqueue(jobs.write_visit_plans_to_data_lake, job_timeout=14800)
utility.log_job_enqueued(logging.logger(), "visit_plan_tasks", job)

2023-10-10 19:51:12,291 <no request_id> : visit_plan_tasks → enqueued job `marketplace.visit_plan.jobs.write_visit_plans_to_data_lake(7dce608e-6e77-4e96-a46e-dd2fff775b02)` for 
INFO:marketplace.logging:visit_plan_tasks → enqueued job `marketplace.visit_plan.jobs.write_visit_plans_to_data_lake(7dce608e-6e77-4e96-a46e-dd2fff775b02)` for 
```

with increased timeout from 7200 to 14800.

