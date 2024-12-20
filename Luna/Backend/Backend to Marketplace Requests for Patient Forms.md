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
	- Checks if the form has a completed visit
	- Checks if the form is needing patient signature
	- Looks for Patient Email related to the form
	- Adds form information to a `results` array


# What tables are involved?

These are the tables living in Marketplace taking a part in the any of the described endpoints.

- patient_form
- integrated_care_plan
- integrated_patient
- patient_email_history

## `patient_form` Table Description

```sql
id uuid DEFAULT public.gen_random_uuid() NOT NULL,
form_type varchar NOT NULL,
created_at timestamptz NOT NULL,
completed_at date NULL,
care_plan_aggregate_id uuid NOT NULL,
status_id uuid NOT NULL,
status_last_checked timestamptz NOT NULL,
questions_url varchar NOT NULL,
answers_url varchar NOT NULL,
form_id varchar NOT NULL,
triggering_appointment_id varchar NULL,
trigger_type int2 DEFAULT '4'::smallint NOT NULL,
CONSTRAINT patient_form_pkey PRIMARY KEY (id)
```

## `integrated_care_plan` Table Description

```sql
id uuid NOT NULL,
patient_aggregate_id uuid NOT NULL,
care_plan_id varchar NOT NULL,
affected_body_part varchar NOT NULL,
surgery_date date NULL,
surgery_form_sent bool NOT NULL,
assigned_physician_npi varchar NULL,
clinic_id varchar NULL,
initial_evaluation_date timestamptz NULL,
CONSTRAINT integrated_care_plan_care_plan_id_key UNIQUE (care_plan_id),
CONSTRAINT integrated_care_plan_pkey PRIMARY KEY (id),
CONSTRAINT integrated_care_plan_id_fkey FOREIGN KEY (id) REFERENCES marketplace."aggregate"(id),
CONSTRAINT integrated_care_plan_patient_aggregate_id_fkey FOREIGN KEY (patient_aggregate_id) REFERENCES marketplace.integrated_patient(id)
```

## `integrated_patient` Table Description

```sql
id uuid NOT NULL,
patient_id varchar NOT NULL,
CONSTRAINT integrated_patient_patient_id_key UNIQUE (patient_id),
CONSTRAINT integrated_patient_pkey PRIMARY KEY (id),
CONSTRAINT integrated_patient_id_fkey FOREIGN KEY (id) REFERENCES marketplace."aggregate"(id)
```

## `patient_email_history` Table Description

```sql
id serial4 NOT NULL,
sent_at timestamptz NOT NULL,
form_id uuid NULL,
care_plan_aggregate_id uuid NOT NULL,
appointment_id varchar NOT NULL,
patient_email_address varchar NOT NULL,
scheduled_time timestamptz NOT NULL,
email_type varchar NOT NULL,
CONSTRAINT patient_email_history_email_type_care_plan_aggregate_id_pat_key UNIQUE (email_type, care_plan_aggregate_id, patient_email_address, appointment_id, scheduled_time),
CONSTRAINT patient_email_history_pkey PRIMARY KEY (id),
CONSTRAINT patient_email_history_care_plan_aggregate_id_fkey FOREIGN KEY (care_plan_aggregate_id) REFERENCES marketplace."aggregate"(id),
CONSTRAINT patient_email_history_form_id_fkey FOREIGN KEY (form_id) REFERENCES marketplace.patient_form(id)
```

## What tables need to be Logical Replicated to Edge?