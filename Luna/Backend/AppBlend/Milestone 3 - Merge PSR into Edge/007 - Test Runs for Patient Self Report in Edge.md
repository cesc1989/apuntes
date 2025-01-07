# Tests runs for Patient Self Report in Edge

Estos son todos los tests para correr de Patient Self Report una vez mezclados en Edge. Pongo este listado aquí para que sea más fácil de encontrar y tener los comandos a la mano.

## Carpeta por Carpeta

En orden según la carpeta `spec`:
```bash
pruebas spec/forms/patient_self_report/

pruebas spec/lib/patient_self_report/
pruebas spec/lib/section_builder/
pruebas spec/lib/webhooks/completed_form_spec.rb

pruebas spec/models/patient_self_report/

pruebas spec/requests/patient_self_report/api/v1/
pruebas spec/requests/patient_self_report/api/v2/
pruebas spec/requests/patient_self_report/api/v3/

pruebas spec/services/patient_self_report/
```

## En Total

Todos en una sola línea:
```bash
pruebas spec/forms/patient_self_report/ spec/lib/patient_self_report/ spec/lib/section_builder/ spec/lib/webhooks/completed_form_spec.rb spec/models/patient_self_report/ spec/requests/patient_self_report/api/v1/ spec/requests/patient_self_report/api/v2/ spec/requests/patient_self_report/api/v3/ spec/services/patient_self_report/
```

## Patrón "form" en paralelo

```bash
be rake "parallel:spec[form]"
```

## Con variable TEST_FILES del CI

Esta es la forma en que el CI elige y corre grupos de pruebas:
```bash
export TEST_FILES="./spec/util/readers/schedule_spec.rb ./spec/queries/proactive_communication/potential_auto_discharge_query_spec.rb ./spec/models/concerns/candid_encounter_data_modifier_spec.rb ./spec/models/patient_self_report/intake_form_spec.rb ./spec/models/patient_invoice_item_spec.rb ./spec/requests/patient_self_report/api/v2/patient_onboarding_statuses_spec.rb ./spec/graphql/authorization/patient_spec.rb ./spec/workers/mass_practice_chart_fax_worker_spec.rb ./spec/requests/patient_self_report/api/v1/onboarding_drafts_spec.rb ./spec/models/patient_self_report/form_type_spec.rb ./spec/requests/admin/payer_options_spec.rb ./spec/workers/proactive_communication/unresolved_visit_ticket_worker_spec.rb ./spec/controllers/api/v1/external/stripe/invoices_controller_spec.rb ./spec/serializers/patient_appointment_serializer_spec.rb ./spec/requests/new_world/graphql/mutations/outbound_comms/fax_plans_of_care_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_validate_procedures_form_spec.rb ./spec/services/therapist_launchpad_auto_disable_service_spec.rb ./spec/models/protocol_physician_spec.rb ./spec/requests/graphql/performance/production_queries_spec.rb ./spec/workers/therapist_tip_charge_worker_spec.rb ./spec/workers/chart_practice_transmitter_worker_spec.rb ./spec/queries/patient_filters/scope_filter_spec.rb ./spec/models/excluded_credentialing_entry_spec.rb ./spec/requests/patient_self_report/api/v3/forms/quality_of_life_spec.rb ./spec/services/plans_of_care/payer/generator_spec.rb ./spec/requests/graphql/mutations/echo/echo_auto_chart_update_subjective_three_spec.rb ./spec/workers/plan_of_care_refax_worker_spec.rb ./spec/controllers/api/v1/patient/injuries_controller_spec.rb ./spec/controllers/api/v1/patient/notifications_manager_controller_spec.rb ./spec/seeders/protocol_seeder_spec.rb ./spec/requests/graphql/mutations/omni/omni_apply_action_add_patient_availabilities_spec.rb ./spec/workers/therapist_1099_emailer_worker_spec.rb ./spec/workers/admin_user_download_worker_spec.rb ./spec/controllers/admin/concierge/patient_no_sessions_report_spec.rb ./spec/models/waitlist_entry_spec.rb ./spec/services/physiotec_service_spec.rb ./spec/models/clinical_escalation_question_spec.rb ./spec/integrations/inbound_integration_base/impl_spec.rb ./spec/models/reviewer_spec.rb"

be rspec $TEST_FILES
```

## Paralelo especificando carpetas

```bash
parallel_rspec -- -f progress -- spec/forms/patient_self_report/ spec/lib/patient_self_report/ spec/lib/section_builder/ spec/lib/webhooks/completed_form_spec.rb spec/models/patient_self_report/ spec/requests/patient_self_report/api/v1/ spec/requests/patient_self_report/api/v2/ spec/requests/patient_self_report/api/v3/ spec/services/patient_self_report/
```