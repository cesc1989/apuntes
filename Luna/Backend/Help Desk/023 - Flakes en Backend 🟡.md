# 023 - Flakes en Backend CI

Etiquetas: #luna_help_desk 

Ryan, Anthony, Meredith y yo hemos visto varios tests que están bien _flekis_. Voy a arreglarlos.

# Cómo correr pruebas

> [!Info]
> Cómo correr pruebas en paralelo para poder probar estos caso ver en [[007 - Test Runs for Patient Self Report in Edge#Patrón "form" en paralelo]]

> [!Tip]
> ¿Cómo correr los tests del bloque del CI en paralelo?
>
> ```
> export TEST_FILES=""
> parallel_rspec -- -f progress -- $TEST_FILES
> ```

# Todos los Reportes

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
```


## models/invoicing_run_spec.rb:21 🟢

> [!Info]
> Estas pruebas ya no existen más porque Ryan las borró en un PR posterior.

Reporte: https://github.com/lunacare/backend/actions/runs/19880118239/job/56976032678?pr=12841


## functional_outcome_measures_service_spec.rb:73 🟢

Prueba
```
rspec ./spec/services/functional_outcome_measures_service_spec.rb:73
```
> FunctionalOutcomeMeasuresService Using multiple form examples reads the applicable form submissions relative to the chart submission date

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
```

## v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:20 🟡

> [!Info]
> El problema de este test es que las questions no se están creando con un orden asignado. Para los FormType ASES se crean 11 preguntas. 10 con option choices y 1 sin. Al crearlas, no se está asignando el atributo `order` así que cuando se quiere buscar una pregunta con option choices el sistema puede devolver una que tiene o una que no.
>
> Para solucionar se busca una que tenga y se selecciona esa.

Prueba:
```
rspec ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:20
```
> Progress Form Drafts GET /progress_form_drafts when question component has selected_option_id shows the selected option choice id

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
```

Fallo adicional:
```
Failures:

  1) Progress Form Drafts GET /progress_form_drafts when question component has selected_option_id shows the selected option choice id
     Got 2 failures:

     1.1) Failure/Error: expect(question[:data]).to include(:selected_option_id)

            expected {"body" => "On average, how much shoulder pain have you experienced in the last week?", "label" => "Your Pain Scale"} to include :selected_option_id
            Diff:
            @@ -1 +1,2 @@
            -[:selected_option_id]
            +"body" => "On average, how much shoulder pain have you experienced in the last week?",
            +"label" => "Your Pain Scale",
          # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:46:in `block (4 levels) in <top (required)>'
          # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:7:in `block (2 levels) in <top (required)>'
          # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
          # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'

     1.2) Failure/Error: expect(question[:data][:selected_option_id]).not_to be(nil)

            expected not #<NilClass:4> => nil
                     got #<NilClass:4> => nil

            Compared using equal?, which compares object identity.
          # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:47:in `block (4 levels) in <top (required)>'
          # ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb:7:in `block (2 levels) in <top (required)>'
```


## models/credentialing/therapist_spec.rb 🟡

Prueba:
```
rspec ./spec/models/credentialing/therapist_spec.rb:60 # Credentialing::Therapist Validations when signing up from New Jersey is expected to validate that :employer_name cannot be empty/falsy
```

Y:
```
rspec ./spec/models/credentialing/therapist_spec.rb:79 # Credentialing::Therapist Instance methods #registered_from? when therapist registered from California state registered from California state
```

Fallos:
```
2) Credentialing::Therapist Validations when signing up from New Jersey is expected to validate that :employer_name cannot be empty/falsy
   Failure/Error: expect(new_jersey_therapist).to validate_presence_of(:employer_name)

     Expected Credentialing::Therapist to validate that :employer_name cannot
     be empty/falsy, but this could not be proved.
       After setting :employer_name to ‹""›, the matcher expected the
       Credentialing::Therapist to be invalid, but it was valid instead.
   # ./spec/models/credentialing/therapist_spec.rb:61:in `block (4 levels) in <top (required)>'
```

Y:
```
3) Credentialing::Therapist Instance methods #registered_from? when therapist registered from California state registered from California state
   Failure/Error: expect(california_therapist).to be_registered_from("california")
     expected `#<Credentialing::Therapist id: "9aab9990-3657-44fd-9c5b-e576f3fc1151", first_name: "Abelardo", phone_...nil, backfilled_attestation_form_url: false, returning_status: "default", signup_date: "2018-10-14">.registered_from?("california")` to be truthy, got false
   # ./spec/models/credentialing/therapist_spec.rb:80:in `block (5 levels) in <top (required)>'
```

## models/exercise_spec.rb 🟢

Prueba:
```
pruebas ./spec/models/exercise_spec.rb
```

Fallo:
```
Failures:

  1) Exercise.times_assigned should be readonly
     Failure/Error: expect(@exercise.reload.times_assigned).to eql 0

       expected: 0
            got: 999

       (compared using eql?)
     # ./spec/models/exercise_spec.rb:21:in `block (3 levels) in <top (required)>'
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
```

Reproducir:
```
bundle exec rspec --order random \
                  --format progress \
                  --seed 31470 \
                  $TEST_FILES
```

## v1/medicare_requirements_spec.rb:35 🟡

Reporte: https://github.com/lunacare/backend/actions/runs/20071009599/job/57576988745?pr=12920

Prueba:
```
rspec ./spec/requests/credentialing/api/v1/medicare_requirements_spec.rb:35
```

Fallo:
```
1.1) Failure/Error: expect(medicare_requirement.answers.where(tc_question_id: 1).count).to eq(1)

       expected: 1
            got: 0

       (compared using ==)
     # ./spec/requests/credentialing/api/v1/medicare_requirements_spec.rb:68:in `block (4 levels) in <top (required)>'

1.2) Failure/Error: expect(medicare_requirement.answers.where(tc_question_id: 3).count).to eq(1)

       expected: 1
            got: 2

       (compared using ==)
     # ./spec/requests/credentialing/api/v1/medicare_requirements_spec.rb:70:in `block (4 levels) in <top (required)>'
```

Correr:
```
parallel_rspec -- -f progress --seed 14670 -- $TEST_FILES
```


## models/patient_self_report/patient_form_helpers_spec.rb:27 🟡

Reporte: https://github.com/lunacare/backend/actions/runs/20079110723/job/57601477442?pr=12920

Prueba:
```
rspec ./spec/models/patient_self_report/patient_form_helpers_spec.rb:27
```
> PatientSelfReport::PatientFormHelpers.get_all_patient_forms_episode_and_status returns two forms, one completed and one incomplete with the right attributes

Fallos:
```
Failures:

  1) PatientSelfReport::PatientFormHelpers.get_all_patient_forms_episode_and_status returns two forms, one completed and one incomplete with the right attributes
     Got 2 failures:

     1.1) Failure/Error:
                  expect(result.first).to eq(
                    id: form_1.id,
                    care_plan_id: form_1.care_plan_id,
                    is_completed: form_1.completed,
            
                    completed_at: form_1.completed_at,
                    created_at: form_1.created_at,
            
                    form_type: form_1.form_type,
                    injury_name: form_1.injury_name,

            expected: {:care_plan_id=>"dc48e053-9b5a-4068-837e-25b52d565c27", :completed_at=>2018-10-14 11:00:00.000000000 +0000, :created_at=>2018-10-12 11:00:00.000000000 +0000, :form_type=>#<PatientSelfReport::FormType id: 31, name: "ASES", acronym: "ases", created_at: "2025-12-09 21:38:48.454020000 +0000", updated_at: "2025-12-09 21:38:48.454020000 +0000">, :id=>5, :injury_name=>"InjuryName5", :is_completed=>true, :progress_type=>"onboarding", :url=>"/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477", :v3_url=>"/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477"}
                 got: {:care_plan_id=>"252a699a-db2c-482b-8727-ca38e3bf4b32", :completed_at=>nil, :created_at=>2018-10-13 11:00:00.000000000 +0000, :form_type=>#<PatientSelfReport::FormType id: 31, name: "ASES", acronym: "ases", created_at: "2025-12-09 21:38:48.454020000 +0000", updated_at: "2025-12-09 21:38:48.454020000 +0000">, :id=>6, :injury_name=>"InjuryName6", :is_completed=>false, :progress_type=>"onboarding", :url=>"/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac", :v3_url=>"/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac"}

            (compared using ==)

            Diff:


            @@ -1,10 +1,10 @@
            -:care_plan_id => "dc48e053-9b5a-4068-837e-25b52d565c27",
            -:completed_at => 2018-10-14 11:00:00.000000000 +0000,
            -:created_at => 2018-10-12 11:00:00.000000000 +0000,
            +:care_plan_id => "252a699a-db2c-482b-8727-ca38e3bf4b32",
            +:completed_at => nil,
            +:created_at => 2018-10-13 11:00:00.000000000 +0000,
             :form_type => #<PatientSelfReport::FormType id: 31, name: "ASES", acronym: "ases", created_at: "2025-12-09 21:38:48.454020000 +0000", updated_at: "2025-12-09 21:38:48.454020000 +0000">,
            -:id => 5,
            -:injury_name => "InjuryName5",
            -:is_completed => true,
            +:id => 6,
            +:injury_name => "InjuryName6",
            +:is_completed => false,
             :progress_type => "onboarding",
            -:url => "/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477",
            -:v3_url => "/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477",
            +:url => "/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac",
            +:v3_url => "/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac",
          # ./spec/models/patient_self_report/patient_form_helpers_spec.rb:32:in `block (3 levels) in <top (required)>'
          # ./spec/models/patient_self_report/patient_form_helpers_spec.rb:7:in `block (2 levels) in <top (required)>'
          # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
          # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'

     1.2) Failure/Error:
                  expect(result.last).to eq(
                    id: form_2.id,
                    care_plan_id: form_2.care_plan_id,
                    is_completed: form_2.completed,
            
                    completed_at: form_2.completed_at,
                    created_at: form_2.created_at,
            
                    form_type: form_2.form_type,
                    injury_name: form_2.injury_name,

            expected: {:care_plan_id=>"252a699a-db2c-482b-8727-ca38e3bf4b32", :completed_at=>nil, :created_at=>2018-10-13 11:00:00.000000000 +0000, :form_type=>#<PatientSelfReport::FormType id: 31, name: "ASES", acronym: "ases", created_at: "2025-12-09 21:38:48.454020000 +0000", updated_at: "2025-12-09 21:38:48.454020000 +0000">, :id=>6, :injury_name=>"InjuryName6", :is_completed=>false, :progress_type=>"onboarding", :url=>"/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac", :v3_url=>"/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac"}
                 got: {:care_plan_id=>"dc48e053-9b5a-4068-837e-25b52d565c27", :completed_at=>2018-10-14 11:00:00.000000000 +0000, :created_at=>2018-10-12 11:00:00.000000000 +0000, :form_type=>#<PatientSelfReport::FormType id: 31, name: "ASES", acronym: "ases", created_at: "2025-12-09 21:38:48.454020000 +0000", updated_at: "2025-12-09 21:38:48.454020000 +0000">, :id=>5, :injury_name=>"InjuryName5", :is_completed=>true, :progress_type=>"onboarding", :url=>"/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477", :v3_url=>"/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477"}

            (compared using ==)

            Diff:


            @@ -1,10 +1,10 @@
            -:care_plan_id => "252a699a-db2c-482b-8727-ca38e3bf4b32",
            -:completed_at => nil,
            -:created_at => 2018-10-13 11:00:00.000000000 +0000,
            +:care_plan_id => "dc48e053-9b5a-4068-837e-25b52d565c27",
            +:completed_at => 2018-10-14 11:00:00.000000000 +0000,
            +:created_at => 2018-10-12 11:00:00.000000000 +0000,
             :form_type => #<PatientSelfReport::FormType id: 31, name: "ASES", acronym: "ases", created_at: "2025-12-09 21:38:48.454020000 +0000", updated_at: "2025-12-09 21:38:48.454020000 +0000">,
            -:id => 6,
            -:injury_name => "InjuryName6",
            -:is_completed => false,
            +:id => 5,
            +:injury_name => "InjuryName5",
            +:is_completed => true,
             :progress_type => "onboarding",
            -:url => "/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac",
            -:v3_url => "/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/forms/d8aa2d31-6345-4476-a8dd-21f10ede49ac",
            +:url => "/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477",
            +:v3_url => "/v3/patients/fa449852-dd09-4a1f-a217-31863f1901d7/summary/0be82c04-c595-45da-9cdc-98c6667b1477",
          # ./spec/models/patient_self_report/patient_form_helpers_spec.rb:47:in `block (3 levels) in <top (required)>'
          # ./spec/models/patient_self_report/patient_form_helpers_spec.rb:7:in `block (2 levels) in <top (required)>'
```

Correr pruebas con seed:
```
parallel_rspec -- -f progress --seed 57621 -- $TEST_FILES
```

O este:
```
pruebas --seed 57621 $TEST_FILES
```

# Buscando Flakes 🔎

## De Patient Self Report

Estos son los que Claudio dice pueden ser problemáticos
```
export TEST_FILES="spec/requests/patient_self_report/api/v1/request_progress_forms_spec.rb spec/requests/patient_self_report/api/v1/care_plan_forms_spec.rb spec/requests/patient_self_report/api/v1/forms_spec.rb spec/requests/patient_self_report/api/v1/forms/intertwined_form_deletion_spec.rb spec/requests/patient_self_report/api/v1/forms/onboarding_form_completed_spec.rb spec/requests/patient_self_report/api/v1/forms/ongoing_form_completed_spec.rb spec/requests/patient_self_report/api/v1/form_status_spec.rb spec/requests/patient_self_report/api/v1/ongoing_drafts_spec.rb spec/requests/patient_self_report/api/v1/diseases_spec.rb spec/requests/patient_self_report/api/v1/previous_intake_forms_spec.rb spec/requests/patient_self_report/api/v1/patient_forms_spec.rb spec/requests/patient_self_report/api/v1/pain_spots_spec.rb spec/requests/patient_self_report/api/v1/request_intake_forms_spec.rb spec/requests/patient_self_report/api/v1/previous_progress_forms_spec.rb spec/requests/patient_self_report/api/v1/onboarding_forms_spec.rb spec/requests/patient_self_report/api/v1/intake_forms_spec.rb spec/requests/patient_self_report/api/v1/reset_forms_spec.rb spec/requests/patient_self_report/api/v1/summaries_spec.rb"
```

## De Credentialing

```
export TEST_FILES="spec/requests/credentialing/api/v2/external/hubspot_ssn_webhooks_spec.rb spec/requests/credentialing/api/v1/therapists/signup_date_spec.rb spec/requests/credentialing/api/v1/submit_spec.rb spec/requests/credentialing/api/v1/therapists_spec.rb spec/requests/credentialing/api/v1/personal_references/default_personal_references_spec.rb spec/requests/credentialing/api/v1/professional_history_spec.rb spec/requests/credentialing/api/v1/immunization_spec.rb spec/requests/credentialing/api/v1/medicare_requirements_spec.rb spec/requests/credentialing/api/v1/ping_spec.rb spec/requests/credentialing/api/v1/credentialing_informations_spec.rb spec/requests/credentialing/api/v1/therapists/zip_coverage_spec.rb spec/requests/credentialing/api/v1/payouts_spec.rb spec/requests/credentialing/api/v1/npi_and_caqh_applications_spec.rb spec/requests/credentialing/api/v1/preferences_spec.rb spec/requests/credentialing/api/v2/internal/reference_files_spec.rb"
```

## De Clinical Dashboard

```
export TEST_FILES="spec/models/clinical_dashboard/dashboard_spec.rb spec/models/clinical_dashboard/user_spec.rb spec/models/clinical_dashboard/link_spec.rb"
```