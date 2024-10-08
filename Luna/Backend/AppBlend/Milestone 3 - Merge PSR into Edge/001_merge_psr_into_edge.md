# 001 - Merge Patient Self Report into Edge

This document details the plan to merge the Patient Self Report backend into Edge. It's divided in two milestones: Database Merge and Code Merge.

# Database Merge

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

Before migrating these tables the `outbox` table has to be deleted and the `patients` table adapted so it does not clash with Edge's `patients`table.

## PSR `patients` vs Edge `patients` Tables

Although similar in name, these two tables do not share the same information. PSR `patients` table needs to be migrated with modifications. I see two ways to do this:

A) create a new table `patient_form_details` that will only hold the fields that are unique to the Patient Self Report `patients` table:

- guardian_name
- signature
- accept_terms_and_conditions

Any part of the Patient Self Report domain needing other patient information would grab it by going to the patient's account record.

B) bring Patient Self Report `patients` table as is but prefix it, i.e `psr_patients`. If choosing this way, it'd be good to also prefix all PSR tables with a defined prefix such as in the example before.

Either option would be link to Edge `patients` via the `internal_id` column present in PSR `patients` table.

## Execution Plan

These are the action items to complete this milestone.

- Delete `outbox` table from Patient Self Report database
- Decide how to integrate the PSR `patients` table
	- Depending on the option there'll be prerequisite work
- Have query do the LR from PSR into Edge in Alpha
- Create new migrations in Edge to use the PSR tables
	- using the `if_not_exists` option in the migration
- Query Edge Alpha db
- Replicate in Omega

# Code Merge

After the Patient Self Report database is migrated to Edge, we can start moving code in. This is the proposed order to do so:

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
- rake tasks, documentation

This order considers classes that are needed for others to work and goes up to less needed.

## Namespace

To create clear boundaries between existing Ruby classes and facilitate working in this satellite app I think it's best to keep these incoming classes in a unique folder to act as a namespace.

For example, all incoming model files will live at `app/models/patient_self_report/*.rb`.

By doing this we make it clear that everything under the namespace belongs to the Patient Self Report domain by just taking a glance. Namespacing provides a clear path forward whenever someyone would need to work on this app.

> Edge is a large codebase with many Ruby classes scattered in multiple places. For some parts of the project related files might be difficult to find or reason about as a group. An example of this is the Protocol Escalation models. To make sense of them one needs to look at the schema, wire associations, and understand subtleties in the models.

Same namespacing approach will be followed in the tests folder, i.e for model specs `spec/models/patient_self_report/*_spec.rb`.

This presents the benefit of a) be able to run all test suite for the Patient Self Report satellite app once inside Edge `rspec spec/models/patient_self_report/`; b) get a glance of all tests involved in this domain. Further helping understand what is all about.

**IMPORTANT**

Considering we're merging three backends into Edge (forms, dashboard, and credentialing), it makes more sense to namespace them to simplify the merge itself and set order and organization of closely related code.

## Replace Service calls with Ruby classes invocations

In Edge, there are multiple Ruby classes that make requests to Patient Self Report endpoints. Once the latter is merged into the former we want to change that to be Ruby classes calls.

To do this causing minimum changes possible in Edge, leave Edge classes that communicate to PSR as they are now. Instead, add an intermediate class to acts as a "controller" and returns whatever Edge's classes expect to receive.

By doing this, what changes in Edge is replacing HTTP requests with a Ruby method call. For instance, let's see `PatientFormsService` method `all_patient_forms(patient_id)`. After PSR is completely merged, it should look like this:
```diff
class PatientFormsService
  attr_accessor :params

  def all_patient_forms(patient_id)
    return [] unless patient_self_report_service_enabled?

-   result = self.class.get(
-     "/v2/backend/all-patient-forms",
-     body: { patient_id: patient_id }.to_json
-   )
+   result = self.class.get(:all_patient_forms, patient_id: patient_id)

    raise PatientFormsServiceError, result.inspect unless result.success?

    JSON.parse(result.body, symbolize_names: true)[:forms]
  end
end
```

Before doing this, I have to identify all places in Edge making requests to Patient Self Report endpoints and all places in Patient Self Report doing requests to Edge.

## Execution Plan

- Identify all places where intercommunication takes place.
- Level up Rails version
	- Currently sits at 7.0.4
- Migrate ActiveRecord models and specs.
- Integrate in Alpha to play with Patient Self Report models in a rails console
	- If any model needs a resource outside of the models folder, skip it in tests (`xit`) and take notice to go back to tests when introduced the missing link
- With CI green merge to Omega
- Repeat for the next element in the adaptation list before
- When all components of PSR are present in Edge do the Edge-to-PSR call swap in code

# Attachments

## Patient Self Report ERD Diagram

![[005 - guardian name.png]]