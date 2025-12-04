# 023 - Flakes en Backend CI

Etiquetas: #luna_help_desk 

Ryan, Anthony, Meredith y yo hemos visto varios tests que están bien _flekis_. Voy a arreglarlos.

> [!Info]
> Cómo correr pruebas en paralelo para poder probar estos caso ver en [[007 - Test Runs for Patient Self Report in Edge#Patrón "form" en paralelo]]

## v2/progress_form_drafts_spec.rb:467 🟢

Reporte: https://github.com/lunacare/backend/actions/runs/19717501852/job/56493127343?pr=12822

Prueba:
```bash
pruebas ./spec/requests/patient_self_report/api/v2/progress_form_drafts_spec.rb:467
```
> Progress Form Drafts PUT /progress_form_drafts when draft is for functional limitations and ASES form answer is updated updates the question option choice

Fallo:
```
Failures:

  1) Progress Form Drafts PUT /progress_form_drafts when draft is for functional limitations and ASES form answer is updated updates the question option choice
     Failure/Error: value: first_question.option_choices.last.id

     NoMethodError:
       undefined method `id' for nil:NilClass

                     value: first_question.option_choices.last.id
                                                              ^^^
     # ./spec/requests/patient_self_report/api/v2/progress_form_drafts_spec.rb:455:in `block (5 levels) in <top (required)>'
     # ./spec/requests/patient_self_report/api/v2/progress_form_drafts_spec.rb:462:in `block (5 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:37:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```


## v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:101 🟢

Reporte: https://github.com/lunacare/backend/actions/runs/19718246762/job/56495443869?pr=12824

Prueba:
```
rspec ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:101 # Progress Form Drafts GET /progress_form_drafts when answer does not have content attribute shows the selected option choice id
```

Fallo:
```
Failures:

  1) Progress Form Drafts GET /progress_form_drafts when answer does not have content attribute shows the selected option choice id
     Failure/Error: expect(question[:data]).to include(:selected_option_id)

       expected {"body" => "On average, how much shoulder pain have you experienced in the last week?", "label" => "Your Pain Scale"} to include :selected_option_id
       Diff:
       @@ -1 +1,2 @@
       -[:selected_option_id]
       +"body" => "On average, how much shoulder pain have you experienced in the last week?",
       +"label" => "Your Pain Scale",
     # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:103:in `block (4 levels) in <top (required)>'
     # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:8:in `block (2 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:37:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```


## v3/drafts_spec.rb:252 🟢

Reporte: https://github.com/lunacare/backend/actions/runs/19722978926/job/56508913949?pr=12828

Prueba:
```
rspec ./spec/requests/patient_self_report/api/v3/drafts_spec.rb:252 # V3 Drafts for JSON Forms POST /v3/patients/:patient_id/forms/:uuid/drafts with valid params for Answers saves answers draft
```

Fallo:
```
Failures:

  1) V3 Drafts for JSON Forms POST /v3/patients/:patient_id/forms/:uuid/drafts with valid params for Answers saves answers draft
     Failure/Error: value: first_option_choice.id

     NoMethodError:
       undefined method `id' for nil:NilClass

                     value: first_option_choice.id
                                               ^^^
     # ./spec/requests/patient_self_report/api/v3/drafts_spec.rb:260:in `block (4 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:37:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```


## patient_form_helpers_spec.rb:22 🟢

Reporte: https://github.com/lunacare/backend/actions/runs/19877554877/job/56968441106?pr=12853

Prueba:
```
rspec ./spec/models/patient_self_report/patient_form_helpers_spec.rb:22 # PatientSelfReport::PatientFormHelpers.get_all_patient_forms_episode_and_status returns two forms, one completed and one incomplete with the right attributes
```

Fallo:
```
Failures:

  1) PatientSelfReport::PatientFormHelpers.get_all_patient_forms_episode_and_status returns two forms, one completed and one incomplete with the right attributes
     Got 2 failures:

     1.1) Failure/Error:
                  expect(result.first).to eq({
                    id: form_1.id,
                    care_plan_id: form_1.care_plan_id,
                    is_completed: form_1.completed,
            
                    completed_at: form_1.completed_at,
                    created_at: form_1.created_at,
            
                    form_type: form_1.form_type,
                    injury_name: form_1.injury_name,

            expected: {:care_plan_id=>"be8373f9-e758-42cd-85d7-5c7c21aaf08c", :completed_at=>2018-10-14 11:00:00.000000000 ...l=>"/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/summary/a363d6b9-ef90-4baf-a000-47292d3e4a3d"}
                 got: {:care_plan_id=>"c94e6fde-592a-4f56-a89a-693d612f2aef", :completed_at=>nil, :created_at=>2018-10-13 1...url=>"/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/forms/07c71b14-d62c-46ce-9fe7-d7d63edb63e5"}

            (compared using ==)

            Diff:


            @@ -1,10 +1,10 @@
            -:care_plan_id => "be8373f9-e758-42cd-85d7-5c7c21aaf08c",
            -:completed_at => 2018-10-14 11:00:00.000000000 +0000,
            -:created_at => 2018-10-12 11:00:00.000000000 +0000,
            +:care_plan_id => "c94e6fde-592a-4f56-a89a-693d612f2aef",
            +:completed_at => nil,
            +:created_at => 2018-10-13 11:00:00.000000000 +0000,
             :form_type => #<PatientSelfReport::FormType id: 285, name: "ASES", acronym: "ases", created_at: "2018-10-14 11:00:00.000000000 +0000", updated_at: "2018-10-14 11:00:00.000000000 +0000">,
            -:id => 35,
            -:injury_name => "InjuryName32",
            -:is_completed => true,
            +:id => 36,
            +:injury_name => "InjuryName33",
            +:is_completed => false,
             :progress_type => "onboarding",
            -:url => "/patients/e382454a-2143-42ad-89ed-ef648f3e393a/summary/a363d6b9-ef90-4baf-a000-47292d3e4a3d",
            -:v3_url => "/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/summary/a363d6b9-ef90-4baf-a000-47292d3e4a3d",
            +:url => "/patients/e382454a-2143-42ad-89ed-ef648f3e393a/forms/07c71b14-d62c-46ce-9fe7-d7d63edb63e5",
            +:v3_url => "/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/forms/07c71b14-d62c-46ce-9fe7-d7d63edb63e5",
          # ./spec/models/patient_self_report/patient_form_helpers_spec.rb:27:in `block (3 levels) in <top (required)>'
          # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
          # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'

     1.2) Failure/Error:
                  expect(result.last).to eq({
                    id: form_2.id,
                    care_plan_id: form_2.care_plan_id,
                    is_completed: form_2.completed,
            
                    completed_at: form_2.completed_at,
                    created_at: form_2.created_at,
            
                    form_type: form_2.form_type,
                    injury_name: form_2.injury_name,

            expected: {:care_plan_id=>"c94e6fde-592a-4f56-a89a-693d612f2aef", :completed_at=>nil, :created_at=>2018-10-13 1...url=>"/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/forms/07c71b14-d62c-46ce-9fe7-d7d63edb63e5"}
                 got: {:care_plan_id=>"be8373f9-e758-42cd-85d7-5c7c21aaf08c", :completed_at=>2018-10-14 11:00:00.000000000 ...l=>"/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/summary/a363d6b9-ef90-4baf-a000-47292d3e4a3d"}

            (compared using ==)

            Diff:


            @@ -1,10 +1,10 @@
            -:care_plan_id => "c94e6fde-592a-4f56-a89a-693d612f2aef",
            -:completed_at => nil,
            -:created_at => 2018-10-13 11:00:00.000000000 +0000,
            +:care_plan_id => "be8373f9-e758-42cd-85d7-5c7c21aaf08c",
            +:completed_at => 2018-10-14 11:00:00.000000000 +0000,
            +:created_at => 2018-10-12 11:00:00.000000000 +0000,
             :form_type => #<PatientSelfReport::FormType id: 285, name: "ASES", acronym: "ases", created_at: "2018-10-14 11:00:00.000000000 +0000", updated_at: "2018-10-14 11:00:00.000000000 +0000">,
            -:id => 36,
            -:injury_name => "InjuryName33",
            -:is_completed => false,
            +:id => 35,
            +:injury_name => "InjuryName32",
            +:is_completed => true,
             :progress_type => "onboarding",
            -:url => "/patients/e382454a-2143-42ad-89ed-ef648f3e393a/forms/07c71b14-d62c-46ce-9fe7-d7d63edb63e5",
            -:v3_url => "/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/forms/07c71b14-d62c-46ce-9fe7-d7d63edb63e5",
            +:url => "/patients/e382454a-2143-42ad-89ed-ef648f3e393a/summary/a363d6b9-ef90-4baf-a000-47292d3e4a3d",
            +:v3_url => "/v3/patients/e382454a-2143-42ad-89ed-ef648f3e393a/summary/a363d6b9-ef90-4baf-a000-47292d3e4a3d",
          # ./spec/models/patient_self_report/patient_form_helpers_spec.rb:43:in `block (3 levels) in <top (required)>'
          # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
          # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```


## models/invoicing_run_spec.rb:21 🟢

> [!Info]
> Estas pruebas ya no existen más porque Ryan las borró en un PR posterior.

Reporte: https://github.com/lunacare/backend/actions/runs/19880118239/job/56976032678?pr=12841

Prueba:
```
rspec ./spec/models/invoicing_run_spec.rb:21 # InvoicingRun.with_supported_invoicing_days calculates business days skipping holidays
```

Y:
```
rspec ./spec/models/invoicing_run_spec.rb:30 # InvoicingRun.with_supported_invoicing_days skips multiple consecutive holidays
```


Fallo:
```
Failures:

  1) InvoicingRun.with_supported_invoicing_days calculates business days skipping holidays
     Failure/Error: expect(result.to_date).to eq(Date.new(2024, 12, 26))

       expected: Thu, 26 Dec 2024
            got: Fri, 27 Dec 2024

       (compared using ==)

       Diff:
       @@ -1 +1 @@
       -Thu, 26 Dec 2024
       +Fri, 27 Dec 2024
     # ./spec/models/invoicing_run_spec.rb:27:in `block (3 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'

  2) InvoicingRun.with_supported_invoicing_days skips multiple consecutive holidays
     Failure/Error: expect(result.to_date).to eq(Date.new(2024, 12, 3))

       expected: Tue, 03 Dec 2024
            got: Wed, 27 Nov 2024

       (compared using ==)

       Diff:
       @@ -1 +1 @@
       -Tue, 03 Dec 2024
       +Wed, 27 Nov 2024
     # ./spec/models/invoicing_run_spec.rb:36:in `block (3 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```

## functional_outcome_measures_service_spec.rb:73 🟡

Prueba
```
rspec ./spec/services/functional_outcome_measures_service_spec.rb:73 # FunctionalOutcomeMeasuresService Using multiple form examples reads the applicable form submissions relative to the chart submission date
```

Fallo:
```
Failures:

  1) FunctionalOutcomeMeasuresService Using multiple form examples reads the applicable form submissions relative to the chart submission date
     Failure/Error:
       expect(result).to eql(
         [
           {
             score_label: "PSFS Score",
             score_value_label: "0/10",
             submitted_at: complete_progress_psfs_outcome_1.completed_at.iso8601
           },
           {
             score_label: "Pain Score",
             score_value_label: "5/10",

       expected: [{:score_label=>"PSFS Score", :score_value_label=>"0/10", :submitted_at=>"2025-12-04T08:35:50Z"}, {:score_label=>"Pain Score", :score_value_label=>"5/10", :submitted_at=>"2025-12-04T08:35:51Z"}]
            got: [{:score_label=>"PSFS Score", :score_value_label=>"1/10", :submitted_at=>"2025-12-04T08:35:50Z"}, {:score_label=>"Pain Score", :score_value_label=>"5/10", :submitted_at=>"2025-12-04T08:35:50Z"}]

       (compared using eql?)

       Diff:

       @@ -1,6 +1,6 @@
        [{:score_label=>"PSFS Score",
       -  :score_value_label=>"0/10",
       +  :score_value_label=>"1/10",
          :submitted_at=>"2025-12-04T08:35:50Z"},
         {:score_label=>"Pain Score",
          :score_value_label=>"5/10",
       -  :submitted_at=>"2025-12-04T08:35:51Z"}]
       +  :submitted_at=>"2025-12-04T08:35:50Z"}]
     # ./spec/services/functional_outcome_measures_service_spec.rb:79:in `block (3 levels) in <top (required)>'
     # ./spec/services/functional_outcome_measures_service_spec.rb:13:in `block (2 levels) in <top (required)>'
     # ./spec/services/functional_outcome_measures_service_spec.rb:7:in `block (2 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```

## v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:20 🟡

Prueba:
```
rspec ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:20 # Progress Form Drafts GET /progress_form_drafts when question component has selected_option_id shows the selected option choice id
```

Fallo:
```
Failures:

  1) Progress Form Drafts GET /progress_form_drafts when question component has selected_option_id shows the selected option choice id
     Failure/Error: content: first_choice.label,

     NoMethodError:
       undefined method `label' for nil:NilClass

                 content: first_choice.label,
                                      ^^^^^^
     # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:35:in `block (4 levels) in <top (required)>'
     # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:7:in `block (2 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```