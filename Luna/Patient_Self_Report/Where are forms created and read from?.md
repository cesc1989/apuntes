# Where are forms created and read from?

In this document I'll list all places I can find where intake forms are created and read from.

## Intake Form Creation

Form creation starts from Backend, goes through Marketplace, for it to finally happen in Patient Self Report, to return the final results to Marketplace where it's saved to the database.

This flow provides an overview of this process:

![[50.intakeform.creation.flow.png]]



## Patient Profile

In the patient profile there's a section where the links to the form are displayed.

![[patient.profile.forms.png]]

This all happens in `app/admin/customers/patients.rb` in a panel block.
```ruby
panel "Details" do
	# (...)
	
    form do
        render partial: "info",
            locals: {
                signed_terms_of_service_status: @signed_terms_of_service_status,
                successfully_fetched_forms: @successfully_fetched_forms,
                needs_to_complete_intake_form: @needs_to_complete_intake_form,
                needs_to_complete_progress_form: @needs_to_complete_progress_form,
                latest_intake_form_url: @latest_intake_form_url,
                latest_progress_form_url: @latest_progress_form_url,
                card_last_4: @card_last_4,
                card_type: @card_type
            }
    end

    # (...)
end
```

The html partial is located at `app/views/admin/patients/_info.html.erb`. It does a lot of stuff but we mainly care about the code where these ivars are used:

```ruby
@latest_intake_form_url
@latest_progress_form_url
```

Those variables are loaded with results from making a call to the patient-self-report service. The call is in the same active admin file:
```ruby
if resource.recent_care_plan.present?
  begin
    forms_result = PatientFormsService.new.patient_onboarding_statuses([resource.id]) if Luna.env.live?
end
```

The endpoint of the service es `/v2/backend/patient-onboarding-statuses`.