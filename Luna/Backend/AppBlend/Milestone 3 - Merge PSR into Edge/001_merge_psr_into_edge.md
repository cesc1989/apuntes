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

Although similar in name, these two tables do not share the same information. PSR `patients` table needs to be migrated with modifications. To start the migration we have to:

A) create a new table `patient_form_details` that will only hold the fields that are unique to the Patient Self Report `patients` table:

- guardian_name
- signature
- accept_terms_and_conditions
- internal_id (will act as foreign_key because PSR `patients` table primary key is bigint)

B) bring Patient Self Report `patients` table as is but prefix it, i.e `psr_patients`.

> Any part of the Patient Self Report domain needing other patient information would grab it by going to the patient's account record.

> The link to Edge `patients` is the `internal_id` column present in PSR `patients` table.

## Execution Plan

These are the action items to complete this milestone.

- Delete `outbox` table from Patient Self Report database
- In current Patient Self Report database:
	- create new model `patient_forms_details`
	- update associations and controllers
	- update data exports
	- update PDF generation
- Have Ops team do the LR in Alpha
- Create new migrations in Edge to use the PSR tables
	- using the `if_not_exists` option in the migration
- Test PSR models in Edge Alpha
- With Ops team, define how to switch the LR writer
	- Initially it's from PSR to Edge
	- When migration is completed should be the other way
- Replicate in Omega

# Code Merge

After the Patient Self Report database is migrated to Edge, we'll be ready to start code adaptation and migration.

## General Plan

First, upgrade Patient Self Report to the current version Edge sits at (whatever that version is).

Then, update the Edge's Docker image (with Infra team help) to install `wkhtmltopdf`. This is a software used to generate PDF files from HTML templates used in this project.

Continue by adding the intermediate classes that will be used in Edge once PSR is copied in. The goal is that these classes produce the same output as the endpoints they will replace.

Next is to identify all places that need a feature flag to control the switch between the current Patient Self Report service and the one that'll live in Edge. Setup appropriate flags in Edge.

Finally, copying all Patient Self Report code in corresponding folders in Edge repository. After the copy is completed, both PSR version (source and Edge) should be working and producing same outputs when tested.

## Namespace

To create clear boundaries between existing Ruby classes and facilitate working in this satellite app I think it's best to keep these incoming classes in a unique folder to act as a namespace.

For example, all incoming model files will live at `app/models/patient_self_report/*.rb`.

By doing this we make it clear that everything under the namespace belongs to the Patient Self Report domain by just taking a glance. Name-spacing provides a clear path forward whenever someone would need to work on this app.

> Edge is a large codebase with many Ruby classes scattered in multiple places. For some parts of the project related files might be difficult to find or reason about as a group. An example of this is the Protocol Escalation models. To make sense of them one needs to look at the schema, wire associations, and understand subtleties in the models.

Same names-pacing approach will be followed in the tests folder, i.e for model specs `spec/models/patient_self_report/*_spec.rb`.

This presents the benefit of a) be able to run all test suite for the Patient Self Report satellite app once inside Edge `rspec spec/models/patient_self_report/`; b) get a glance of all tests involved in this domain. Further helping understand what is all about.

**IMPORTANT**

Considering we're merging three backends into Edge (forms, dashboard, and credentialing), it makes more sense to namespace them to simplify the merge itself and set order and organization of closely related code.

## Routes

Split routes.rb into a separate file.

Bring all of the routes from Patient Self Report into its own file and include it in the main Edge’s `routes.rb`.

By doing this we do not clutter anymore this file and define clear boundaries between subsystems inside Edge.

```ruby
# config/initializers/routing_draw.rb
class ActionDispatch::Routing::Mapper
  def draw(routes_name)
    instance_eval(File.read(Rails.root.join("config/routes/#{routes_name}.rb")))
  end
end

# config/routes/patient_self_report.rb
namespace :patient_self_report do
  # routes...
end

# config/routes.rb
Rails.application.routes.draw do
  draw :patient_self_report
end
```

## Replace Service calls with Ruby classes invocations

In Edge, there are multiple Ruby classes that make requests to Patient Self Report endpoints. Once the latter is merged into the former we want to change that to be Ruby classes calls.

To do this causing minimum changes possible in Edge, leave Edge classes that communicate to PSR as they are now. Instead, add an intermediate class to acts as a "controller" and returns whatever Edge's classes expect to receive.

By doing this, what changes in Edge is replacing HTTP requests with a Ruby method call. For instance, let's see `PatientFormsService` method `all_patient_forms(patient_id)`. After PSR is completely merged, it should look like this:
```diff
class PatientFormsService
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

- Identify all places where intercommunication takes place
- Level up Rails version
- Create all Ruby classes that will act as intermediaries
- Copy, in appropriate namespaces, all PSR code into Edge
	- The goal here is to be able to run both PSR in its current env and it Edge
	- Both version should produce the same output when fetching data or using endpoints
- Define date and time for Infra team to switch off old PSR and switch on new version inside Edge backend

# Attachments

## Patient Self Report ERD Diagram

![[005 - guardian name.png]]