# Clinical Dashboard User Email Verification

These are (most) the classes involved in this process:

- PortalProviderEntity
- ShadowUser  
- CommunicationMethods
- UserCommunicationMethods::Email
- UserCommunicationMethods::EmailVerificationWorker

Send/re-send email verification link

- app/admin/customers/physicians.rb
- admin::EmailVerificationLinksController
- app/views/admin/physicians/_escalation_email_recipient_verification_information.html.haml

Mailer:

- UserCommunicationMethods::EmailVerificationMailer
- verification_email.html.erb
- translations in this view are in `user_communication_methods.en.yml`

HTML view of the landing page:

- UserCommunicationMethods::EmailVerificationsController
- email_verifications/new.html.haml (landing page)
- UserCommunicationMethods::Base

Files in requesting links from Clinical Dashboard

- ProviderPortalService

## Why does the link last 72 hours?

In alpha, it is only valid for 72 hours. In Omega it depends if there's custom setting else is the same amount of hours.

# Manual and automated tests

To verify changes to the landing page at `app/views/user_communication_methods/email_verifications/new.html.haml` I can manually test them by opening the page at `localhost:3000/email_verification/SOMETKKEN`. Comment out the conditionals to see the different paths.

Or automated tests at `spec/mailers/user_communication_methods/email_verification_mailer_spec.rb` for the mailer sent.
