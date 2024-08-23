# AWS Athena Production Queries
Queries using aggregation clauses like `GROUP BY` or `COUNT` won’t work with the filtering from the UI because there’s no way to send Athena a parameter to the `WHERE` clause.

In the **Patients Treated** table, the filtering went OK in the first try because the result set for that query is *just a list of rows* where every row has the name of the physician.

The queries that have some sort of grouping or counting don’t have the result set in that way and as **it’s not possible to send parameters to Athena via the Ruby SDK***, we need to have a query that returns the results as in the **Patients Treated** table and do the grouping/counting in the backend and after that, do the filtering.


> *It is possible to [arrange the query string](https://docs.aws.amazon.com/athena/latest/APIReference/API_StartQueryExecution.html#API_StartQueryExecution_RequestSyntax) sent to Athena but it won’t be as fast as needed
# Normalized Queries

**CASE DISTRIBUTION (1) ✅** 
Query name: *PROD_Normalized_CD*

    /*PROD_Normalized_Case_Distribution*/
    SELECT forms.injury_name,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**QUALITY OF LIFE (2) ✅** 
Query name: *PROD_Normalized_QoL*

    /*PROD_Normalized_Quality_of_Life*/
    SELECT forms.quality_of_life,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE forms.quality_of_life IS NOT NULL
    AND forms.quality_of_life <> ''
    AND pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**AGE DISTRIBUTION (3) ✅** 
Query name: *PROD_Normalized_AD*

    /*PROD_Normalized_Age_Distribution*/
    SELECT CAST(pat.age AS bigint) AS age,
      pat.gender,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name
    FROM "luxe-production-db"."patients" pat
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**VISITS BY INJURY TYPE (4) ✅** 
Query name: *PROD_Normalized_VBIT*

    /*PROD_Normalized_Visits_By_Injury_Type*/
    SELECT forms.injury_name,
      pat.completed_visits_count,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**CHANGE IN PAIN LEVEL 5/10 VISITS (5) ✅** 
Query name: *PROD_Normalized_CIPL*

    /*PROD_Normalized_Change_In_Pain_Level*/
    SELECT forms.injury_name,
      forms.pain_0,
      forms.pain_1,
      forms.pain_2,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE forms.pain_0 IS NOT null
    AND pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**CHANGE IN PSFS SCALE 5/10 VISITS (6) ✅** 
Query name: *PROD_Normalized_CIPS*

    /*PROD_Normalized_Change_In_Psfs_Scale*/
    SELECT forms.injury_name,
      forms.psfs_0,
      forms.psfs_1,
      forms.psfs_2,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE forms.psfs_0 IS NOT null
    AND pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**BODY PARTS (7) ✅** 
Query name: *PROD_Normalized_BP*

    /*PROD_Normalized_Body_Parts*/
    SELECT forms.injury_name,
      forms.form_type,
      forms.pain_0, forms.pain_1,
      forms.psfs_0, forms.psfs_1,
      forms.answers_0, forms.answers_1,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      cli.name AS partner_name,
      forms.pain_2, forms.psfs_2,
      forms.pain_3, forms.psfs_3,
      forms.pain_4, forms.psfs_4,
      forms.pain_5, forms.psfs_5,
      forms.pain_6, forms.psfs_6,
      forms.pain_7, forms.psfs_7,
      forms.pain_8, forms.psfs_8,
      forms.pain_9, forms.psfs_9,
      forms.answers_2, forms.answers_3, forms.answers_4, forms.answers_5,
      forms.answers_6, forms.answers_7, forms.answers_8, forms.answers_9
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE forms.injury_name IN ('Knee', 'Hip', 'Lower Back', 'Shoulder/Arm', 'Neck', 'Knee - Joint Replacement', 'Hip - Joint Replacement', 'Upper back')
    AND pat.latest_visit_date > to_iso8601(current_date - interval '90' day);

**RECENT PATIENTS TREATED (8) ✅** 
Query name: *PROD_Normalized_PT*

    /*PROD_Normalized_Patients_Treated*/
    SELECT pat.first_name,
      CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      pat.completed_visits_count, pat.pending_visits_count,
      forms.pain_0, forms.psfs_0,
      forms.pain_1, forms.psfs_1,
      forms.pain_2, forms.psfs_2,
      forms.pain_3, forms.psfs_3,
      forms.pain_4, forms.psfs_4,
      forms.pain_5, forms.psfs_5,
      forms.pain_6, forms.psfs_6,
      forms.pain_7, forms.psfs_7,
      forms.pain_8, forms.psfs_8,
      forms.pain_9, forms.psfs_9,
      pat.discharged, forms.injury_name, forms.form_type,
      forms.answers_0, forms.answers_1, forms.answers_2, forms.answers_3, forms.answers_4,
      forms.answers_5, forms.answers_6, forms.answers_7, forms.answers_8, forms.answers_9,
      pat.last_name,
      cli.name AS partner_name,
      pat.id AS patient_id,
      phy.id AS physician_id
    FROM "patient-forms-production-db"."production" forms
    JOIN "luxe-production-db"."patients" pat ON pat.id = forms.internal_id
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."partner_clinics" cli ON cli.id = pat.partner_clinic_id
    WHERE pat.latest_visit_date > to_iso8601(current_date - interval '90' day)
    ORDER BY pat.last_name;

**PHYSICIANS LIST (9) ✅** 
Query name: *PROD_Normalized_PL*

    /*PROD_Normalized_Physicians_List*/
    SELECT CONCAT(phy.first_name, ' ', phy.last_name) AS physician_name,
      gro.name AS group_name,
      phy.prefix AS physician_prefix,
      phy.id AS physician_id
    FROM "luxe-production-db"."patients" pat
    LEFT JOIN "luxe-production-db"."physicians" phy ON phy.id = pat.physician_id
    LEFT JOIN "luxe-production-db"."physician_groups" gro ON gro.id = phy.physician_group_id
    WHERE pat.latest_visit_date > to_iso8601(current_date - interval '90' day)
    GROUP BY phy.first_name,
      phy.prefix,
      CONCAT(phy.first_name, ' ', phy.last_name),
      gro.name
    ORDER BY phy.first_name;


# Queries for Permissions

**For Physician Permissions ✅** 
Query name: *PROD_Normalized_Phy_P*

    /*PROD_Normalized_Physicians_Permissions*/
    SELECT DISTINCT
      first_name,
      last_name,
      can_sign_patient_pocs,
      can_view_patient_charts,
      can_view_assigned_physicians
    FROM "luxe-production-db"."physicians"
    ORDER BY first_name ASC;

**For Clinic Group Permissions ✅** 
Query name: *PROD_Normalized_CGP*

    /*PROD_Normalized_Clinic_Groups_Permissions*/
    SELECT DISTINCT
      name,
      can_sign_patient_pocs,
      can_view_patient_charts,
      can_view_assigned_physicians
    FROM "luxe-production-db"."physician_groups"
    ORDER BY name ASC;

**For Partner Permissions ✅** 
Query name: *PROD_Normalized_Part_P*

    /*PROD_Normalized_Partner_Permissions*/
    SELECT DISTINCT
      name,
      code,
      can_sign_patient_pocs,
      can_view_patient_charts,
      can_view_assigned_physicians
    FROM "luxe-production-db"."partner_clinics"
    ORDER BY name ASC;


# Queries Without Normalization 🤚🏼 🛑 

**CASE DISTRIBUTION**
Query name: *PROD_Case_Distribution*

    /*PROD_Case_Distribution*/
    SELECT injury_name, physician_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND latest_visit_date > to_iso8601(current_date - interval '90' day);

**QUALITY OF LIFE**
Query name: *PROD_Quality_of_Life*

    /*PROD_Quality_of_Life*/
    SELECT quality_of_life, physician_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND quality_of_life IS NOT NULL
    AND quality_of_life <> ''
    AND latest_visit_date > to_iso8601(current_date - interval '90' day);

**AGE DISTRIBUTION**
Query name: *PROD_Age_Distribution*

    /*PROD_Age_Distribution*/
    SELECT age, gender, physician_name
    FROM "luxe-production-db"."production"
    WHERE latest_visit_date > to_iso8601(current_date - interval '90' day);

**VISITS BY INJURY TYPE**
Query name: *PROD_Visits_By_Injury_Type*

    /*PROD_Visits_By_Injury_Type*/
    SELECT injury_name, completed_visits_count, physician_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND latest_visit_date > to_iso8601(current_date - interval '90' day);

**CHANGE IN PAIN LEVEL 5/10 VISITS**
Query name: *PROD_Change_In_Pain_Level*

    /*PROD_Change_In_Pain_Level*/
    SELECT injury_name, pain_0, pain_1, pain_2, physician_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND pain_0 is NOT null
    AND latest_visit_date > to_iso8601(current_date - interval '90' day);

**CHANGE IN PSFS SCALE 5/10 VISITS**
Query name: *PROD_Change_In_PSFS_Scale*

    /*PROD_Change_In_PSFS_Scale*/
    SELECT injury_name, psfs_0, psfs_1, psfs_2, physician_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND psfs_0 is NOT null
    AND latest_visit_date > to_iso8601(current_date - interval '90' day);

**BODY PARTS**
Query name: *PROD_Body_Parts*

    /*PROD_Body_Parts*/
    SELECT injury_name, form_type,
           pain_0, pain_1,
           psfs_0, psfs_1,
           answers_0, answers_1, physician_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND injury_name IN ('Knee', 'Hip', 'Lower Back', 'Shoulder/Arm', 'Neck', 'Hip - Joint Replacement', 'Knee - Joint Replacement')
    AND latest_visit_date > to_iso8601(current_date - interval '90' day);

**RECENT PATIENTS TREATED**
Query name: *PROD_Patients_Treated*

    /*PROD_Patients_Treated*/
    SELECT first_name, physician_name, completed_visits_count, pending_visits_count,
      pain_0, psfs_0,
      pain_1, psfs_1,
      pain_2, psfs_2,
      pain_3, psfs_3,
      pain_4, psfs_4,
      pain_5, psfs_5,
      pain_6, psfs_6,
      pain_7, psfs_7,
      pain_8, psfs_8,
      pain_9, psfs_9,
      discharged, injury_name, form_type,
      answers_0, answers_1, answers_2, answers_3, answers_4,
      answers_5, answers_6, answers_7, answers_8, answers_9,
      last_name
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND latest_visit_date > to_iso8601(current_date - interval '90' day)
    ORDER BY last_name;

**PHYSICIANS LIST**
Query name: *PROD_Physicians_List*

    /*PROD_Physicians_List*/
    SELECT physician_name, physician_group, physician_prefix
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND latest_visit_date > to_iso8601(current_date - interval '90' day)
    GROUP BY physician_name, physician_group, physician_prefix
    ORDER BY physician_name;
# Queries With Grouping 🤚🏼 🛑 

**CASE DISTRIBUTION (grouping)**

    SELECT injury_name, count(injury_name) as injury_count
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    GROUP BY injury_name
    ORDER BY injury_count DESC

**QUALITY OF LIFE (grouping)**

    SELECT
      quality_of_life,
      count(quality_of_life) AS qol_count
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    AND quality_of_life IS NOT NULL
    AND quality_of_life <> ''
    GROUP BY quality_of_life
    ORDER BY  quality_of_life ASC

**VISITS BY INJURY TYPE (grouping)**

    WITH completed_visits_by_injury AS (
    SELECT injury_name, completed_visits_count,
      CASE
        WHEN completed_visits_count < 5 THEN '<5'
        WHEN completed_visits_count > 10 THEN '>10'
        ELSE '5-10'
      END AS completed_visits_bucket
    FROM "luxe-production-db"."production" b, "patient-forms-production-db"."production" f
    WHERE b.id = f.internal_id
    )
    
    SELECT injury_name, completed_visits_bucket, count() as num_patients
    FROM completed_visits_by_injury
    GROUP BY injury_name, completed_visits_bucket
    ORDER BY injury_name, completed_visits_bucket

**AGE DISTRIBUTION (grouping)**

    SELECT age_range, gender, count() as num_patients
    from (
      SELECT case
        when age between 0 and 9 then '0-9'
        when age between 10 and 19 then '10-19'
        when age between 20 and 29 then '20-29'
        when age between 30 and 39 then '30-39'
        when age between 40 and 49 then '40-49'
        when age between 50 and 59 then '50-59'
        when age between 60 and 69 then '60-69'
        when age between 70 and 79 then '70-79'
        when age between 80 and 89 then '80-89'
        when age between 90 and 99 then '90-99'
        else '100+' end as age_range,
        gender
      FROM "luxe-production-db"."production"
    )
    GROUP BY age_range, gender
    ORDER BY gender, age_range DESC

