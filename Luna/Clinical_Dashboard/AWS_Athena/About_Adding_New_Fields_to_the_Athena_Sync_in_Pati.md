# About Adding New Fields to the Athena Sync in Patient Self Report
**tl; dr.**
The order in which we add fields to the CSV dump in Patient Forms does matter.

## Exporting `care_plan_id`

When I worked in sending the `care_plan_id` value to Athena, the first attempt was just [adding in the middle](https://github.com/lunacare/patient-forms-backend/pull/661/commits/35f845a46561cac588b0834b107dde28b9cbd940) of the whole list of fields:

      completed_forms_count: forms.where(completed: true).count,
      form_type: first_form.type_name,
      care_plan_id: first_form.care_plan_id
    }.merge(scores(forms))

However, that broke query results in Athena because Glue schema definition depends heavily in the CSV order.

In the end, I [had to do](https://github.com/lunacare/patient-forms-backend/pull/661/commits/a08e4fb7e63b8cc63ab64b4fd193a79dc8d6c1c6) this:

    export = {
      id: patient.id,
      internal_id: patient.internal_id,
      quality_of_life: latest_intake_form.quality_of_life,
      injury_name: first_form.injury_name,
      forms_count: forms.count,
      completed_forms_count: forms.where(completed: true).count,
      form_type: first_form.type_name
    }.merge(scores(forms))
    
    export.merge(care_plan_id: first_form.care_plan_id)

So that `care_plan_id` would be that last field in the CSV and also the last one in Glue.

If I wanted the `care_plan_id` to be somewhere in the middle and also match it to the Glue schema, then I’ve had to recreate the schema in Glue as there’s no way to reorder the list of fields.

## Exporting `progress_modality_(0..9)`

When doing the change to export the field `submitted_from` to Athena, the hash looks like this:

    {
      "pain_#{i}": form&.pain_scale,
      "psfs_#{i}": form&.psfs_score,
      "answers_#{i}": form&.score,
      "completed_at_#{i}": form&.completed_at&.iso8601,
      "created_at_#{i}": form&.created_at&.iso8601,
      "progress_modality_#{i}": form&.submitted_from
    }

Now that Athena and Glue configs are terraformed, the schema places the fields with the `_(0..9)` suffix in the numerical order:

    pain_0
    psfs_0
    answers_0
    completed_at_0
    created_at_0
    progress_modality_0

This is good and bad.

**The good part(s):**

- the queries just work because the position in the CSV matches the position in the Glue schema.
- It is a simple change in the AthenaDataSync class.

**The bad part(s):**

> The bad parts are mostly because the change is in staging and I have to be careful it doesn’t go to prod without solving next issues.


- If I leave it like this, I’ll have to manually recreate the schema for the production environment as we haven’t yet Terraformed the configs for prod.
- 

