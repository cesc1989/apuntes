# ✅ Spike: Rename patients table for Logical Replication in Provider Portal
*Update March 19th: No need to do this. Use Edge’s patients table.*

----------
## About

The efforts to complete “Remove Athena” in Provider Portal include setting up Logical Replication of all involved tables from Patient Self Report and Luxe databases into the Provider Portal database.

These two projects happen to have a “patients” table that need to be replicated in Provider Portal database. These two tables cannot exist at the same time in the same database. To finish the Logical Replication setup the Patient Self Report patients table needs to be renamed.

In this document, I describe steps to do this and considerations of things that might be affected by this change.

## Questions
- How to rename the table?
- Is there any test coverage to be done?
- What migrations reference the “patients” table?
- Where is the “patients” table being used directly?
- What are the implications of doing this rename?
# How to rename the table?

Write a rename migration:

    rename_table :patients, :patient_summaries

Use new table name in Patient model:

    class Patient < ApplicationRecord
      self.table_name = "patient_summaries"
    end
# Is there any test coverage to be done?

Coverage is at 83.1%. Any file that touches the patient model is covered enough. No need to add more coverage.

# What migrations reference the “patients” table?

In order:

1. CreatePatients
2. CreateFormRequests
3. CreateIntakeForms
4. RenameSignatureLinkInPatients
5. ChangeInternalIdTypeInPatients
6. AddBirthdayAndGenderToPatients
7. AddEmailToPatient
8. AddGuardianNameToPatient
# Where is the “patients” table being used directly?

The patients table is not used directly in any place across the repository.

# What are the implications of doing this rename?
## On Data Integrity

Renaming the table won't affect the data stored in it, but bear in mind foreign keys, indexes, or constraints remain intact after the rename. Update those accordingly.

**Existing Indices related to patients**
Forms on patient_id

    t.index ["patient_id"], name: "index_forms_on_patient_id"

Intake forms on patient_id

    t.index ["patient_id"], name: "index_intake_forms_on_patient_id"

Patients on internal_id

    t.index ["internal_id"], name: "index_patients_on_internal_id", unique: true

**Existing Foreign keys to patients**
In forms table

    add_foreign_key "forms", "patients"

In intake forms table

    add_foreign_key "intake_forms", "patients"
## On Migrations

Migrations referencing the “patients” table need to be updated. However, when setting up the project from scratch it is not recommended to run `rails db:migrate` but `rails db:setup`. The latter command will load the database from the `db/schema.rb` file which is the most up to date representation of the whole database.

Using `rails db:setup` is the recommended way in the README to setup the project. Once the table is renamed, we’d need to make sure it still works as intended.

## On External Systems

There are several external services that make requests to Patient Self Report. Those services do not interact directly with the “patients” table the rename shouldn’t be an issue for them.

