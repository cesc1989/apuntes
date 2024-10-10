# Upgrade Edge to 7.0.8.4

# ActionView::MissingTemplate in Admin::PatientDuplicates#compare

En Patient Duplicates al clicar "Compare & Resolve".

```
Showing /Users/francisco/projects/luna-project/backend/app/views/admin/patient_duplicates_report/compare.html.erb where line #26 raised:

Missing partial admin/patient_duplicates/_patient_information, active_admin/resource/_patient_information, active_admin/base/_patient_information, inherited_resources/base/_patient_information, application/_patient_information with {:locale=>[:en], :formats=>[:html], :variants=>[], :handlers=>[:raw, :erb, :html, :builder, :ruby, :coffee, :arb, :haml]}.

Searched in:
  * "/Users/francisco/projects/luna-project/backend/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/rails_email_preview-2.2.3/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/graphiql-rails-1.9.0/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/devise_token_auth-1.2.1/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/devise-security-0.17.0/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/devise-4.8.1/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/activeadmin-2.14.0/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/kaminari-core-1.2.2/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/actiontext-7.0.8.4/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/actionmailbox-7.0.8.4/app/views"

Did you mean?  admin/patient_duplicates_report/patient_information
               active_admin/base/bonus_form
               admin/patients/patient_outstanding_items
               admin/patients/waive_form
               admin/patients/info
               admin/patients/edit_appointment_button
```

En
```ruby
<div class="patient-dup-container">
    <div class="p1 patient-container <%= "selected" if suggested_patient(@patient1, @patient2) == @patient1 %>">
      <%= render "patient_information", patient: @patient1 %>
      <% unless antimatch_exists? || merge_exists? || !can_merge?(@patient1, @patient2) %>
        <div class="patient-info-section">
          <label class="patient-info-header">Actions</label>
```