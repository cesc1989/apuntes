# Progress Form not created after Copying Intake Form

Etiquetas: #luna_help_desk 

> Looking at the manage forms link, the intake form was completed at 05/29 and _was copied from care plan 1 to care plan 2 (due to injury type change)_. However, the Progress Form which should have become available (and incomplete/blank) after visit #4 (06/09/24) is not present.

## Relevant Documents

- [[Copy Forms Logic]]


## Related PRs

Latest to oldest:

- Fix Missing Progress Forms on Intake form copying (closed) - [Link](https://github.com/lunacare/backend/pull/11821)
- Fix Form copy not including intake form details (merged) - [Link](https://github.com/lunacare/backend/pull/11702)
- Replace copy_form_data_from_form (merged) - [Link](https://github.com/lunacare/backend/pull/11078)
- Enable moving all forms between care plans (merge) - [Link](https://github.com/lunacare/backend/pull/11019)

## 1st Hypothesis: The `completed_at` value not being updated in Marketplace after Intake Form copy 🟡

These values:
```ruby
target_form.completed = source_form.completed
target_form.completed_at = source_form.completed_at
```

are copied from the completed Intake Form to the incomplete one. However, the same value is not reflected to the TARGET Intake Form in Luxe.

### Verification: check Marketplace TARGET Intake Form completed_at is null

Check for Intake Form with uuid: `6bfc64b5-d57f-4e55-830e-dfedcf680419`.

This query returns no value for `completed_at`:
```sql
select pf.form_type,
 pf.form_id,
 pf.created_at,
 pf.completed_at,
 pf.care_plan_aggregate_id
from patient_form pf
where pf.form_id = '6bfc64b5-d57f-4e55-830e-dfedcf680419'
;
```

|form_type|form_id|created_at|completed_at|care_plan_aggregate_id|
|---------|-------|----------|------------|----------------------|
|intake|6bfc64b5-d57f-4e55-830e-dfedcf680419|2026-01-08 14:16:20.143 -0500||517a81ef-09ef-4be7-9308-be5b93a487e8|
