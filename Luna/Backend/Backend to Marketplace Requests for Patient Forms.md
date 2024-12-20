# What needs to be done to not make calls to Marketplace for all things Patient Forms in Edge?

Now that Patient Forms code lives in Edge we can stop making calls to Marketplace to get information about Forms. What are the things we can stop doing? What is needed to do them?

# What are the endpoints in use?

In `app/services/marketplace.rb` there's two methods that make calls to Marketplace endpoints:

- `get_care_plan_forms`
- `get_intake_form_code_reds`

## get_care_plan_forms

This method makes a GET request to `/api/v1/luxe/care-plan-forms/<care-plan-id>` endpoint in Marketplace. The response is an array of Forms for the given Care Plan ID.

In Marketplace, this endpoint is setup in `app/marketplace/forms/blueprint.py`. This endpoint traverses multiple functions to provide a response.

Functions in `forms/blueprint.py`

- `admin_care_plan_forms_view`
- `get_admin_form_statuses`

Functions in `forms/repository.py`

- `get_all_care_plan_forms`

This last function find all forms (in the `patient_form` table) for the expected care_plan_id(s).

### Form model && `patient_form` table in Marketplace

Marketplace holds a `patient_form` table with information about the created forms. The model describing this table lives in `app/marketplace/es/models.py`, class `Form`.

## get_intake_form_code_reds

This method makes a GET request to `/api/v1/luxe/intake-form-code-red-report` endpoint in Marketplace.

In Marketplace, this endpoint is setup in `app/marketplace/forms/blueprint.py`. This endpoint traverses multiple functions to provide a response.

Functions in `forms/blueprint.py`

- `therapist_view_intake_form_code_reds`
- `get_form_code_reds`

This last function find all forms flagged as "Code Red". Key events in this function:

- Find all incomplete Intake Forms
- For each form:
	- Find its care plan
	- Get care plan's active appointments
	- Checks if the form as a completed visit
	- Checks if the form is needed patient signature
	- Looks for Patient Email related to the form
	- Adds form information to a `results` array


# What tables are involved?

These are the tables living in Marketplace taking a part in the any of the described endpoints.

- patient_form
- integrated_care_plan
- integrated_patient
- patient_email_history