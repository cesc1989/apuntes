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

## Mailers

The intake form links is used in mailers and notifications. Let's see some places and actions.

### resend forms link email

Once again in the active admin `patients.rb` file we can find this snippet:
```ruby
member_action :resend_intake_form, method: :post do
    PatientMailer.resend_intake_form(resource, params[:intake_form_url]).deliver_now

    redirect_back fallback_location: admin_patient_path(resource),
                  notice: "Sent schedule reminder email w/ link to intake form!"
  end
```

And for progress forms:
```ruby
member_action :resend_progress_form, method: :post do
    resource.send_progress_form_reminders({
      form_url: params[:progress_form_url],
      reminder_type: "resend",
      form_type: "progress"
    })

    redirect_back fallback_location: admin_patient_path(resource),
                  notice: "Sent schedule reminder w/ link to progress form!"
  end
```

This takes us to two mailer actions:

- resend_intake_form
- resend_progress_form

but they don't matter. I need to know where they're grabbing the link from. The answer to this question lays in the file `app/views/admin/patients/_actions.html.erb`.

Snippet:
```ruby
<% if needs_to_complete_intake_form  %>
      <div class="grid__item admin_btn" style="white-space: nowrap;">
        <%= link_to "Resend Intake Form", resend_intake_form_admin_patient_path(resource, intake_form_url: latest_intake_form_url), method: :post %>
        <span><% "Last Sent #{resource.intake_email_last_sent_at&.strftime('%m/%d/%Y') || 'Never'}" %></span>
      </div>
    <% elsif needs_to_complete_progress_form %>
      <div class="grid__item admin_btn" style="white-space: nowrap;">
        <%= link_to "Resend Progress Form", resend_progress_form_admin_patient_path(resource, progress_form_url: latest_progress_form_url), method: :post %>
        <span><% "Last Sent #{resource.progress_reminder_emailed_at&.strftime('%m/%d/%Y') || 'Never'}" %></span>
      </div>
    <% end %>
```

Which takes me back to ivars defined previously but passed as locals.
```ruby
latest_intake_form_url
latest_progress_form_url
```

## Manage Forms subview

This view is available from the patients profile.

![[forms.center.png]]

It shows all forms and provides a link to the summary page. The file where the code is located is at `app/views/admin/patients/manage_forms.html.arb`

Several things happen here.

First there's a request to the patient self report service:

```ruby
begin
  @patient_forms = PatientFormsService.new.all_patient_forms(resource.id)
rescue PatientFormsServiceError => e
  error = e
  ExceptionLogger.log(e, "Error fetching patient forms: #{e}")
end
```

This service requests to the endpoint `/v2/backend/all-patient-forms`. The response is divided and sorted in different groups but the url is accessed through JSON response `url` attribute:

```ruby
form[:url]
```

This attribute is defined in the `edge_management_form_serializer.rb` serializer class in Patient Self Report.