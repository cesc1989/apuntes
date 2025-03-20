# 001 - Merge Clinical Dashboard into Edge

This document details the plan to merge the Patient Self Report backend into Edge. It's divided in three milestones: Level Up, Database Merge and Code Merge.

# Level Up

## Upgrade Ruby to 3.1.6

Current version is 3.1.0.

## Upgrade Rails to 7.1.4

Current version is 7.0.4.

## Upgrade Gems

Due to multiple gems upgrades in previous milestones, some gems might have been upgraded. Find those that exists in the Clinical Dashboard project and level the up.

## Execution Plan

- Upgrade CD to Ruby 3.1.0
- Upgrade Gems
- Upgrade CD to Rails 7.0.8.4
- Upgrade CD to Rails 7.1.4
- Enjoy

# Database Merge

The Clinical Dashboard database consists of 25 tables. However, only 6 tables correspond to the actual Clinical Dashboard. The rest were copied for a Logical Replication setup that was never completed.

These are the tables for the Clinical Dashboard project itself:

- activities
- dashboards
- execution_ids
- jwt_denylist (*can be deleted if empty in Omega*)
- links
- pending_plan_of_cares (*can be deleted*)
- users

These are all the tables copied for the incomplete LR setup:

- accounts
- appointments
- auto_charts
- charts
- clinics
- episodes
- forms
- intake_forms
- patient_summaries
- patients
- physician_groups
- physicians
- portal_configs
- practices
- regions
- scores
- therapists
- tins

These tables must be dropped. Corresponding existing models must be deleted.

## Rename `users` Tables

These tables are used to give access to some systems. For this milestone, rename the Clinical Dashboard `users` table to `dashboard_users` to distinguish it from Edge `admin_users`.

The `users` table is only used to provide access to the Dashboard's Admin view. They log in using a email-password combo.

> [!Note]
> In a Post Release stage, records in the `users` table could be migrated the `admin_users`.

## Execution Plan

- Delete all unused tables
	- Remove existing models if any
- Rename `users` table to `dashboard_users`
	- Rename model and test logins
- Write new migrations in Edge to host Clinical Dashboard tables
	- using the `if_not_exists` option in the migration
- Ask Ops team to setup LR in Alpha: CD -> Edge
	- This is going to be temporary until this milestone is released

# Code Merge

After the Clinical Dashboard database is migrated to Edge, we'll be ready to start code migration and adaptation.

## Environment Variables

These envs need to be copied over to Edge. Let's follow the same process as in Milestone 3b:

- Rename similar value envs to match's the name they have in Edge
- Apply the new name in the current repo
- Remove any unused/unneeded env

```bash
HUBSPOT_ACCESS_TOKEN=""

AWS_REGION=""
AWS_ACCESS_KEY_ID=""
AWS_SECRET_ACCESS_KEY=""

S3_ACCESS_ID=""
S3_SECRET_KEY=""
S3_AWS_REGION=""
DATA_LAKE_BUCKET_NAME=""

DOMAIN="http://localhost:3000"
PHYSICIAN_PORTAL_TOKEN=""

UNSIGNED_PLAN_OF_CARE_BUCKET=""
UNSIGNED_CHARTS_BUCKET=""

EDGE_API_DOMAIN=""
EDGE_API_TOKEN=""
KONG_EDGE_PROVIDER_PORTAL_TOKEN=""

CHART_DOWNLOADS_BUCKET=""
PATIENT_CSV_DOWNLOAD_BUCKET=""

RECAPTCHA_SITE_KEY=""
RECAPTCHA_SECRET_KEY=""

JWT_EXPIRATION_SECONDS_LINK_RECIPIENT=500
JWT_EXPIRATION_SECONDS_ADMIN_USER=1200
```

## Namespace

Following what was done in the Milestone 3b, we'll introduce all Ruby code into a `clinical_dashboard` namespace in every folder where migrated code is placed.

## Routes

As it was done in Milestone 3b, split routes.rb into a separate file.

Bring all of the routes from Clinical Dashboard into its own file and include it in the main Edge’s `routes.rb`.

By doing this we do not clutter anymore this file and define clear boundaries between subsystems inside Edge.

```ruby
# config/initializers/routing_draw.rb
class ActionDispatch::Routing::Mapper
  def draw(routes_name)
    instance_eval(File.read(Rails.root.join("config/routes/#{routes_name}.rb")))
  end
end

# config/routes/clinical_dashboard.rb
namespace :clinical_dashboard do
  # routes...
end

# config/routes.rb
Rails.application.routes.draw do
  draw :clinical_dashboard
end
```

## Replace Service calls with Ruby classes invocations

> [!Note]
> To be completed in a Post Release stage.

There are multiple places where requests to Edge endpoints take place. Once Clinical Dashboard is migrated it'll be making requests to itself.

This already happened in Milestone 3b and it's ok. However, we should plan and schedule work to replace all those requests to use direct Ruby calls.

### Remove v2/links route and controller

The endpoint would no longer be needed to generate Portal links. Instead, Edge code should call `LinkGenerationV2Service` class.

### Change SharedChartsService to use Edge models instead

This class would not need to make a request to Edge but use the models to produce the expected response.

In Edge, the route `api/v3/physician_dashboard/physicians/patients/charts` can be removed. The controller `ChartsController` under same namespace can be removed and code moved into a PORO class.

### Change PendingPlansOfCareService to use Edge models instead

This class would not need to make a request to Edge but use the models to produce the expected response.

In Edge, the route `api/v3/physician_dashboard/physicians/unsigned_plans_of_care` can be removed. The controller `UnsignedPlansOfCareController` under same namespace can be removed and code moved into a PORO class.

### Change PlanOfCareSigner to use Edge models instead

This class would not need to make a request to Edge but use the models to produce the expected response.

In Edge, the route `api/v3/physician_dashboard/physicians/plans_of_care/plan_of_care_actions` can be removed. The controller `PlanOfCareActionsController` under same namespace can be removed and code moved into a PORO class.

## Copy Docs Folder

All information that helps understand the Clinical Dashboard. How it works and how is everything related.e

## Execution Plan

- Rename and copy ENV vars with Ops help
- Copy routes file
- Copy docs folder
- Copy all of the code!
- Make tests pass
- QA Dashboard