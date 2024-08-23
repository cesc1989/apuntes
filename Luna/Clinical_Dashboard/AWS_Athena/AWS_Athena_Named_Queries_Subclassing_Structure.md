# AWS Athena Named Queries Subclassing Structure
This is the subclassing structure that the Athena SDK returns in every response for `Aws::Athena::Client.new.get_named_query`.

**This is useful to me to see how to mock requests to Athena in RSpec tests**.

## Case Distribution Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5da389b08
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5da389a40
            database = "patient-forms-production-db",
            description = "Production Case Distribution no Grouping",
            name = "PROD_Case_Distribution",
            named_query_id = "c8238d6b-a89a-4939-bede-f670d341fd67",
            query_string = "SELECT injury_name, physician_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)",
            work_group = "primary"
        >
    >
## Quality of Life Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5da707a78
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5da7079d8
            database = "athena-input-staging-patient-forms",
            description = "Prod Quality of Life No Grouping",
            name = "PROD_Quality_of_Life",
            named_query_id = "0185c476-21e4-4e82-9c84-6b47d15b9a05",
            query_string = "SELECT quality_of_life, physician_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND quality_of_life IS NOT NULL\nAND quality_of_life <> ''\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)",
            work_group = "primary"
        >
    >
## Visits by Injury Type Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5dbcdb728
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5dbcdb6d8
            database = "athena-input-staging-patient-forms",
            description = "Production Visits by Injury Type no Grouping",
            name = "PROD_Visits_By_Injury_Type",
            named_query_id = "c3065c27-02b0-4b15-adf6-097cca653897",
            query_string = "SELECT injury_name, completed_visits_count, physician_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)\n",
            work_group = "primary"
        >
    >
## Age Distribution Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5dd0b4c18
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5dd0b4b50
            database = "athena-input-staging-patient-forms",
            description = "Production Age Distribution No Grouping",
            name = "PROD_Age_Distribution",
            named_query_id = "26350ffc-02b3-492f-b494-56b96e617c17",
            query_string = "SELECT age,\ngender,\nphysician_name\nFROM \"luxe-production-db\".\"production\"\nWHERE latest_visit_date > to_iso8601(current_date - interval '90' day)",
            work_group = "primary"
        >
    >
## Change In Pain Level Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5d6d26ac0
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5d6d26a70
            database = "athena-input-staging-patient-forms",
            description = "Production Change in Pain Level 2",
            name = "PROD_Change_In_Pain_Level",
            named_query_id = "a2b4f48c-e9b5-463b-9ff9-7f118371d3e5",
            query_string = "SELECT injury_name, pain_0, pain_1, pain_2, physician_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND pain_0 is NOT null\nAND pain_1 is NOT null\nAND pain_2 is NOT null\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)",
            work_group = "primary"
        >
    >
## Change In PSFS Level Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5dbb450f8
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5dbb450a8
            database = "athena-input-staging-patient-forms",
            description = "Production Change in PSFS Scale 2",
            name = "PROD_Change_In_PSFS_Scale",
            named_query_id = "3858ea91-3c6a-437c-96aa-e476e80270ef",
            query_string = "SELECT injury_name, psfs_0, psfs_1, psfs_2, physician_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND psfs_0 is NOT null\nAND psfs_1 is NOT null\nAND psfs_2 is NOT null\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)",
            work_group = "primary"
        >
    >
## Body Parts Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5da609ef0
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5da609e50
            database = "athena-input-staging-patient-forms",
            description = "Production Body Parts 3",
            name = "PROD_Body_Parts",
            named_query_id = "0a37691e-b0a1-4e61-bbc5-9b6cdf28b014",
            query_string = "SELECT injury_name, form_type,\n       pain_0, pain_1,\n       psfs_0, psfs_1,\n       answers_0, answers_1, physician_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND injury_name IN ('Knee', 'Hip', 'Lower Back', 'Shoulder/Arm', 'Neck')\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)",
            work_group = "primary"
        >
    >
## Physicians List Query
    <#<Class:0x00007fb5d64be4c8>:Aws::Athena::Types::GetNamedQueryOutput:0x7fb5da027988
        named_query = #<#<Class:0x00007fb5d64b42e8>:Aws::Athena::Types::NamedQuery:0x7fb5da027870
            database = "luxe-production-db",
            description = "Production Physicians List 4",
            name = "PROD_Physicians_List",
            named_query_id = "235bc845-d7dd-4e11-841a-24f2dc74b5a5",
            query_string = "SELECT physician_name, physician_group, physician_prefix\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id\nAND latest_visit_date > to_iso8601(current_date - interval '90' day)\nGROUP BY physician_name, physician_group, physician_prefix\nORDER BY physician_name",
            work_group = "primary"
        >
    >
## Patients Treated Query
    <#<Class:0x00007fc8d2f5d078>:Aws::Athena::Types::GetNamedQueryOutput:0x7fc8d2f7cc70
        named_query = #<#<Class:0x00007fc8d2f4ecd0>:Aws::Athena::Types::NamedQuery:0x7fc8d2f7c9f0
            database = "athena-input-staging-patient-forms",
            description = "Production Patients Treated 4",
            name = "PROD_Patients_Treated",
            named_query_id = "9bfc19f2-d19b-4393-8242-39c6f960eed8",
            query_string = "SELECT\n  first_name, physician_name, completed_visits_count, pending_visits_count,\n  pain_0, psfs_0,\n  pain_1, psfs_1,\n  pain_2, psfs_2,\n  pain_3, psfs_3,\n  pain_4, psfs_4,\n  pain_5, psfs_5,\n  pain_6, psfs_6,\n  pain_7, psfs_7,\n  pain_8, psfs_8,\n  pain_9, psfs_9,\n  discharged, injury_name, form_type,\n  answers_0, answers_1, answers_2, answers_3, answers_4,\n  answers_5, answers_6, answers_7, answers_8, answers_9,\n  last_name\nFROM \"luxe-production-db\".\"production\" b, \"patient-forms-production-db\".\"production\" f\nWHERE b.id = f.internal_id AND latest_visit_date > to_iso8601(current_date - interval '90' day)\nORDER BY last_name",
            work_group = "primary"
        >
    >
## Named Query IDs
    <#<Class:0x00007fe11464b840>:Aws::Athena::Types::ListNamedQueriesOutput:0x7fe1193bc1b0
        named_query_ids = [
            [ 0] "235bc845-d7dd-4e11-841a-24f2dc74b5a5",
            [ 1] "0a37691e-b0a1-4e61-bbc5-9b6cdf28b014",
            [ 2] "3858ea91-3c6a-437c-96aa-e476e80270ef",
            [ 3] "a2b4f48c-e9b5-463b-9ff9-7f118371d3e5",
            [ 4] "c3065c27-02b0-4b15-adf6-097cca653897",
            [ 5] "0185c476-21e4-4e82-9c84-6b47d15b9a05",
            [ 6] "c8238d6b-a89a-4939-bede-f670d341fd67",
            # ...
        ],
        next_token = nil
    >


## Start Query Execution
    <#<Class:0x00007fcd90691f08>:Aws::Athena::Types::StartQueryExecutionOutput:0x7fcd9516ea30
        query_execution_id = "01f00d46-354f-47df-bcf8-383abf2e6926"
    >

