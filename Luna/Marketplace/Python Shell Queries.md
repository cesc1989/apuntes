# Python Shell Queries

Queries para encontrar datos desde una python shell en un container.

## Listar Forms de un Patient

```python
from marketplace.database import Session
from marketplace.es.models import IntegratedPatient, IntegratedCarePlan, Form

session = Session()

patient_id = "0bbec751-8cb8-47f5-97c0-2d6b8962abf6"
forms = (session.query(Form)
    .join(IntegratedCarePlan, Form.care_plan_aggregate_id == IntegratedCarePlan.id)
    .join(IntegratedPatient, IntegratedCarePlan.patient_aggregate_id == IntegratedPatient.id)
    .filter(IntegratedPatient.patient_id == patient_id)
    .all())

for form in forms:
    print(f"Form: {form.form_id}, Type: {form.form_type}")
```

