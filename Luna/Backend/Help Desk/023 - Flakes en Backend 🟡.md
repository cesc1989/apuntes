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

Test Group:
```
export TEST_FILES="./spec/requests/patient_self_report/api/v2/progress_form_drafts_spec.rb ./spec/services/patient_balance_summary_calculator_spec.rb ./spec/services/therapist_payout_calculator_spec.rb ./spec/services/metrics_service_spec.rb ./spec/requests/patient_self_report/api/v2/form_outcomes_spec.rb ./spec/workers/athena/mips_writer_worker_spec.rb ./spec/models/clinic_payer_spec.rb ./spec/workers/athena/care_plan_summary_writer_worker_spec.rb ./spec/services/hubspot_sync_contacts_service_spec.rb ./spec/models/document_spec.rb ./spec/workers/invoicing/ancillary/daily_billing_breakdown_slack_worker_spec.rb ./spec/services/auto_chart_interview_complete_service_spec.rb ./spec/requests/api/v1/external/stripe/invoices_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_validate_rom_form_spec.rb ./spec/workers/athena/exercise_program_writer_worker_spec.rb ./spec/workers/invoicing/running/execute_invoicing_target_worker_spec.rb ./spec/controllers/api/v2/therapist/sessions_controller_spec.rb ./spec/requests/patient_self_report/api/v2/form_status/kos_adl_form_status_cursor_spec.rb ./spec/queries/plans_of_care_query_spec.rb ./spec/lib/events/care_plan_messenger_spec.rb ./spec/workers/clear_bookings_worker_spec.rb ./spec/workers/collections/bulk_unpause_all_patients_worker_spec.rb ./spec/requests/graphql/queries/fields/node_support_topic_query_spec.rb ./spec/requests/api/grimoire_bridge/therapists_spec.rb ./spec/models/excluded_credentialing_entry_spec.rb ./spec/grimoire/grimoire/patients/plans_of_care/physician_response_workflow/update_plan_of_care_worker_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_update_objective_three_spec.rb ./spec/services/sendbird_spec.rb ./spec/workers/hubspot/sync_medicare_threshold_exceeded_worker_spec.rb ./spec/requests/admin/stripe_payment_methods_webhooks_spec.rb ./spec/requests/graphql/queries/fields/node_exercise_program_query_spec.rb ./spec/requests/api/v1/external/twilio_user_sms_verifications_spec.rb ./spec/services/create_proactive_ticket_service_spec.rb ./spec/models/practice_payer_plan_booking_user_option_mapping_item_spec.rb ./spec/models/credentialing/npi_and_caqh_application_spec.rb ./spec/controllers/graphql_controller_spec.rb ./spec/requests/graphql/queries/fields/node_region_query_spec.rb ./spec/uploaders/credentialing/s3_uploader_spec.rb ./spec/requests/credentialing/api/v1/therapists/zip_coverage_spec.rb ./spec/models/visit_plan_entry_spec.rb ./spec/serializers/admin/clinic_serializer_spec.rb ./spec/controllers/admin/concierge/patient_no_sessions_report_spec.rb ./spec/services/clinical_dashboard/requesters/change_in_psfs_scale_requester_service_spec.rb ./spec/mailers/admin_user_download_mailer_spec.rb ./spec/workers/credentialing/hubspot_npi_and_caqh_worker_spec.rb ./spec/lib/clinical_dashboard/athena_results_converter/patients_treated_converter_spec.rb ./spec/lib/clinical_dashboard/athena_results_converter/visits_by_injury_converter_spec.rb ./spec/models/invoicing_target_line_item_snapshot_spec.rb ./spec/models/patient_service_fee_payer_plan_category_exclusion_spec.rb ./spec/controllers/api/v1/external/callrail_controller_spec.rb"
```

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

Test Group:
```
export TEST_FILES="./spec/services/collection_phase_event_manager_spec.rb ./spec/services/direct_access_pocs_service_spec.rb ./spec/candid/candid/sync_invoice_itemization_service_spec.rb ./spec/services/plans_of_care/direct_access/message_calculator_spec.rb ./spec/workers/candid/ingress/ingest_patients_invoice_itemizations_worker_spec.rb ./spec/requests/patient_self_report/api/v1/summaries_spec.rb ./spec/requests/api/v1/internal/waitlist_spec.rb ./spec/requests/patient_self_report/api/v2/progress_form_drafts/progress_form_drafts_selected_option_id_spec.rb ./spec/requests/graphql/queries/fields/viewer_patient_query_spec.rb ./spec/workers/billing_routes/refresh_unserviceable_worker_spec.rb ./spec/services/therapist_availability_reminder_sms_service_spec.rb ./spec/requests/graphql/queries/fields/node_care_plan_outcome_forms_query_spec.rb ./spec/workers/chart_clinic_transmitter_worker_spec.rb ./spec/workers/knock_patient_birthday_worker_spec.rb ./spec/workers/cancel_example_patient_appointments_worker_spec.rb ./spec/controllers/api/v2/patient/accounts_controller_spec.rb ./spec/lib/webhooks/completed_form_spec.rb ./spec/requests/patient_self_report/api/v1/intake_forms_spec.rb ./spec/lib/events/reminder_messenger_spec.rb ./spec/workers/enable_auto_accept_patients_worker_spec.rb ./spec/services/therapist_tip_charge_service_spec.rb ./spec/workers/therapist_quarterly_payout_email_worker_spec.rb ./spec/requests/graphql/queries/fields/document_file_url_spec.rb ./spec/models/concerns/npi_entity_spec.rb ./spec/models/happiness_survey_spec.rb ./spec/controllers/reviewer/automation_sessions_controller_spec.rb ./spec/requests/luna_self_service/graphql/queries/fields/search_regions_query_spec.rb ./spec/models/kx_modifier/patient_spec.rb ./spec/requests/graphql/mutations/notifications/silence_notification_spec.rb ./spec/workers/hubspot_sync_referral_worker_spec.rb ./spec/requests/credentialing/api/v1/professional_history_spec.rb ./spec/models/credentialing/npi_and_caqh_application_spec.rb ./spec/queries/patients_query_spec.rb ./spec/models/therapist_specialty_spec.rb ./spec/requests/clinical_dashboard/api/v1/plans_of_care_spec.rb ./spec/requests/patient_self_report/api/v1/pain_spots_spec.rb ./spec/controllers/credit_card_controller_spec.rb ./spec/requests/credentialing/api/v2/external/attestation/immunization_spec.rb ./spec/grimoire/actions/update_patient_hubspot_contact_spec.rb ./spec/models/invoicing_run_spec.rb ./spec/requests/credentialing/api/v2/internal/reference_files_spec.rb ./spec/controllers/incoming_knock_webhook_controller_spec.rb ./spec/uploaders/credentialing/packet_pdf_uploader_spec.rb ./spec/lib/clinical_dashboard/athena_named_query_finder/change_in_pain_level_all_time_spec.rb ./spec/models/clinical_dashboard/user_spec.rb ./spec/workers/credentialing/hubspot_payout_worker_spec.rb ./spec/models/patient_self_report/surgery_spec.rb ./spec/grimoire/outbound_faxing/send_fax_workflow/workers/send_uploaded_fax_worker_spec.rb ./spec/models/reviewer_spec.rb ./spec/workers/athena/candid_unsynced_appointments_writer_worker_spec.rb ./spec/controllers/api/v1/internal/therapists_controller_spec.rb"
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

Test Group:
```
export TEST_FILES="./spec/requests/patient_self_report/api/v2/progress_form_drafts_spec.rb ./spec/workers/athena/app_installs_from_notifications_writer_worker_spec.rb ./spec/workers/invoicing/running/update_stripe_invoice_worker_spec.rb ./spec/requests/clinical_dashboard/api/v1/physicians_spec.rb ./spec/requests/patient_self_report/api/v3/drafts_spec.rb ./spec/controllers/api/v3/patient/workouts_controller_spec.rb ./spec/models/account_spec.rb ./spec/requests/patient_self_report/api/v2/progress_form_drafts/pagination_spec.rb ./spec/requests/graphql/queries/fields/viewer_patient_query_spec.rb ./spec/services/plans_of_care/workers_comp/violation_calculator_spec.rb ./spec/workers/athena/care_plan_summary_writer_worker_spec.rb ./spec/grimoire/actions/book_initial_visit_spec.rb ./spec/workers/billing_routes/refresh_servicing_worker_spec.rb ./spec/integration/routes_spec.rb ./spec/services/payer_plan_alias_resolver_spec.rb ./spec/candid/candid/mappers/candid_encounter_data_diff_spec.rb ./spec/models/patient_self_report/patient_form_helpers_spec.rb ./spec/requests/patient_self_report/api/v1/intake_forms_spec.rb ./spec/workers/invoicing/ancillary/backfill_collection_events_worker_spec.rb ./spec/workers/patient_email_update_worker_spec.rb ./spec/queries/poc_filters/scope_filter_spec.rb ./spec/services/therapist_insta_pay_payout_service_spec.rb ./spec/services/icd10cm_search_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_update_objective_two_spec.rb ./spec/requests/graphql/mutations/scheduling/add_patient_availability_spec.rb ./spec/workers/collections_patients_time_keeper_worker_spec.rb ./spec/requests/credentialing/api/v2/external/attestation/credentialing_informations_spec.rb ./spec/workers/badges/calculate_badges_worker_spec.rb ./spec/services/clinical_dashboard/physicians_json_builder_service_spec.rb ./spec/services/sendbird_spec.rb ./spec/requests/luna_self_service/graphql/queries/fields/search_regions_query_spec.rb ./spec/candid/candid/client_spec.rb ./spec/workers/waystar_html_loader_worker_spec.rb ./spec/grimoire/grimoire/care_plans/assign_referral_workflow_spec.rb ./spec/models/patient_self_report/question_spec.rb ./spec/requests/api/v1/external/stripe/invoice_items_spec.rb ./spec/services/business_hours_service_spec.rb ./spec/requests/graphql/mutations/grimoire_clear_patient_availabilities_spec.rb ./spec/models/concerns/invoicing_logging_spec.rb ./spec/models/referral_spec.rb ./spec/requests/credentialing/api/v2/external/hubspot_ssn_webhooks_spec.rb ./spec/controllers/admin/money/uncharged_appointments_report_spec.rb ./spec/controllers/inactive_session_controller_spec.rb ./spec/services/search/tokens_parser_spec.rb ./spec/lib/clinical_dashboard/athena_named_query_finder/partners_permissions_spec.rb ./spec/models/invoicing_target_line_item_snapshot_spec.rb ./spec/lib/clinical_dashboard/athena_results_converter/change_in_pain_level_converter_spec.rb ./spec/models/clinical_pathway_spec.rb ./spec/lib/clinical_dashboard/athena_results_converter/visits_by_injury_converter_spec.rb ./spec/models/deduction_spec.rb ./spec/models/injury_spec.rb ./spec/controllers/api/v3/safety_shield_sms_controller_spec.rb"
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

Test Group:
```
export TEST_FILES="./spec/requests/rack_attack/throttling_spec.rb ./spec/requests/admin/customers/therapists_spec.rb ./spec/forms/patient_self_report/admission_form_spec.rb ./spec/models/concerns/candid_encounter_data_modifier_spec.rb ./spec/services/kx_modifier/medicare_dollar_threshold_status_refresher_service_spec.rb ./spec/workers/invoicing/ancillary/fix_collections_pause_data_worker_spec.rb ./spec/graphql/authorization/patient_spec.rb ./spec/controllers/concerns/account_devices_spec.rb ./spec/services/care_plan_status_check_graphql_service_spec.rb ./spec/requests/api/grimoire_bridge/patients/plans_of_care/physician_responses_spec.rb ./spec/workers/wilted_tree_reminder_worker_spec.rb ./spec/models/document_share_spec.rb ./spec/requests/graphql/mutations/scheduling/update_patient_availabilities_spec.rb ./spec/requests/graphql/mutations/therapists/submit_outcome_exclusions_spec.rb ./spec/controllers/api/v1/patient/conversations_controller_spec.rb ./spec/controllers/api/v1/external/workouts_controller_spec.rb ./spec/requests/patient_self_report/api/v2/form_summaries_spec.rb ./spec/models/patient_self_report/patient_form_helpers_spec.rb ./spec/workers/invoicing/ancillary/re_enable_bill_via_invoice_worker_spec.rb ./spec/controllers/draft_therapist_controller_spec.rb ./spec/queries/poc_filters/chart_appointment_visit_type_filter_spec.rb ./spec/requests/api/admin/therapist_searches_spec.rb ./spec/queries/poc_filters/region_filter_spec.rb ./spec/services/time_off_normalization_service_spec.rb ./spec/services/plans_of_care/direct_access/generator_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_update_plan_spec.rb ./spec/models/setting_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_validate_objective_five_spec.rb ./spec/requests/graphql/mutations/protocols/submit_care_pathway_assessment_spec.rb ./spec/requests/api/v1/external/exercises_spec.rb ./spec/workers/care_plan_refresh_proposed_clinic_worker_spec.rb ./spec/models/stripe_payment_intent_spec.rb ./spec/models/therapist_specialty_spec.rb ./spec/candid/candid/client_spec.rb ./spec/requests/graphql/queries/fields/node_document_query_spec.rb ./spec/services/patient_forms_signed_terms_service_spec.rb ./spec/requests/credentialing/api/v1/therapists/zip_coverage_spec.rb ./spec/controllers/api/v1/external/knock/message_sent_controller_spec.rb ./spec/models/patient_serviceability_check_spec.rb ./spec/services/credentialing/hubspot_credentialing_service_spec.rb ./spec/models/referral_spec.rb ./spec/serializers/admin/account_serializer_spec.rb ./spec/lib/clinical_dashboard/athena_named_query_finder/visits_by_injury_all_time_spec.rb ./spec/services/clinical_dashboard/requesters/clinic_groups_permissions_requester_service_spec.rb ./spec/models/document_view_spec.rb ./spec/models/credentialing/medicare_requirement_spec.rb ./spec/workers/outbox_events/cleanup_worker_spec.rb ./spec/models/billing_filter_clinic_spec.rb ./spec/models/temporary_code_spec.rb ./spec/services/age_calculator_spec.rb ./spec/workers/athena/provider_portal_partner_clinic_writer_worker_spec.rb ./spec/requests/graphql/queries/authorization_query_spec.rb"
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
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
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
     # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
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
          # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
          # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
          # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
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
app_1       | 
     Expected Credentialing::Therapist to validate that :employer_name cannot
     be empty/falsy, but this could not be proved.
       After setting :employer_name to ‹""›, the matcher expected the
       Credentialing::Therapist to be invalid, but it was valid instead.
   # ./spec/models/credentialing/therapist_spec.rb:61:in `block (4 levels) in <top (required)>'
   # ./spec/rails_helper.rb:200:in `block (3 levels) in <top (required)>'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/modifier.rb:29:in `run_block'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/modifier.rb:12:in `block in process'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/environment.rb:21:in `block in synchronize'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/environment.rb:19:in `synchronize'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/environment.rb:19:in `synchronize'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/modifier.rb:10:in `process'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control.rb:10:in `modify'
   # ./spec/support/environment_helpers.rb:6:in `with_modified_env'
   # ./spec/rails_helper.rb:197:in `block (2 levels) in <top (required)>'
   # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
   # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
app_1       |
```

Y:
```
3) Credentialing::Therapist Instance methods #registered_from? when therapist registered from California state registered from California state
   Failure/Error: expect(california_therapist).to be_registered_from("california")
     expected `#<Credentialing::Therapist id: "9aab9990-3657-44fd-9c5b-e576f3fc1151", first_name: "Abelardo", phone_...nil, backfilled_attestation_form_url: false, returning_status: "default", signup_date: "2018-10-14">.registered_from?("california")` to be truthy, got false
   # ./spec/models/credentialing/therapist_spec.rb:80:in `block (5 levels) in <top (required)>'
   # ./spec/rails_helper.rb:200:in `block (3 levels) in <top (required)>'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/modifier.rb:29:in `run_block'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/modifier.rb:12:in `block in process'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/environment.rb:21:in `block in synchronize'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/environment.rb:19:in `synchronize'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/environment.rb:19:in `synchronize'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control/modifier.rb:10:in `process'
   # /usr/local/bundle/gems/climate_control-1.0.1/lib/climate_control.rb:10:in `modify'
   # ./spec/support/environment_helpers.rb:6:in `with_modified_env'
   # ./spec/rails_helper.rb:197:in `block (2 levels) in <top (required)>'
   # ./spec/support/active_record_logger.rb:38:in `block (2 levels) in <top (required)>'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
   # /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
   # /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
```