# Clinical Dashboard User Email Verification

These are (most) the classes involved in this process:

- PortalProviderEntity
- ShadowUser  
- CommunicationMethods
- app/admin/customers/physicians.rb
- admin::EmailVerificationLinksController
- UserCommunicationMethods::Email
- UserCommunicationMethods::EmailVerificationWorker
- UserCommunicationMethods::EmailVerificationMailer

HTML view of the landing page:

- verification_email.html.erb
- translations in this view are in `user_communication_methods.en.yml`

The verification occurs in:

- UserCommunicationMethods::EmailVerificationsController
- email_verifications/new.html.haml
- UserCommunicationMethods::Base

## Why does the link last 72 hours?

In alpha, it is only valid for 72 hours. In Omega it depends if there's custom setting else is the same amount of hours.