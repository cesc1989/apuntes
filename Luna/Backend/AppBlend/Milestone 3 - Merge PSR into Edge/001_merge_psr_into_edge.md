# 001 - Merge Patient Self Report into Edge

This document details the plan to merge the Patient Self Report backend into Edge. It's divided in two main blocks: Database merge and Code merge.

# Patient Self Report Database Merge

To do:
- Diagram both models. Individually and altogether??

The Patient Self Report database consists of 15 tables:
- aggravating_activities
- answers
- diseases
- form_types
- forms
- intake_form_diseases
- intake_form_pain_spots
- intake_forms
- medications
- option_choices
- outbox
- pain_spots
- patients
- questions
- settings
- surgeries

Before migrating these tables the `outbox` table have to be deleted and adapt the `patients` table so that it does not clash with Edge's `patients`table.

## PSR `patients` vs Edge `patients` Tables

I see two ways to do this:

A) create a new table `patient_form_details` that will only hold the fields that are unique to the Patient Self Report `patients` table:

- guardian_name
- signature
- accept_terms_and_conditions

B) bring Patient Self Report `patients` table as is but prefix it, i.e, `forms_patients`. 

Either option would be link to Edge `patients` via the `internal_id` column.

# Patient Self Report Code Merge & Services Adaptation

Consider:
- Namespacing every rails domain thing (models, services, controllers, routes)
- Do not alter Edge's calls to PSR but put a controller in between

After the Patient Self Report database is migrated to Edge, we can start moving code. This is the proposed order to do this:

- models
- routes
- assets
	- these are for PDF generation
- lib folder
- services
- controllers & serializers
- views & uploaders
	- install `wkhtmltopdf` software and gems
- workers, helpers, exceptions

This order considers classes that are needed for others to work and goes up to less needed.

## Namespace

To create clear boundaries between Ruby classes and facilitate working in this satellite app I think it's best to keep this incoming classes in a unique folder to act as a namespace.

For example, all incoming model files will live at `app/models/patient_self_report/*.rb`. By doing this we make it clear that everything under the namespace belongs to the Patient Self Report domain and will provide a clear path forward whenever anyone would need to work on this app.

> Edge is a large codebase with many Ruby classes scattered in multiple places. For some parts of the project related files might be difficult to find or reason about as group. An example of this is the Protocol Escalation models. The only way to make sense of them is by looking at the schema and build the associations.


## Replace Service calls with Ruby classes invocations

