# User Email Verification - General Info

This verification is done for access to Clinical Dashboard and Patient Escalations.

These are some of classes involved in this process:

- PortalProviderEntity
- ShadowUser  
- CommunicationMethods (Concern)
- UserCommunicationMethods::Email
- UserCommunicationMethods::EmailVerificationWorker

Send/re-send email verification link

- app/admin/customers/physicians.rb
- Admin::EmailVerificationLinksController
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

## ⭐️ Flow for sending / resending link ⭐️

A `UserCommunicationMethod` instance receives the `send_verification!` or the  `resend_verification!` messages.

```ruby
# Admin::EmailVerificationLinksController

email = UserCommunicationMethod.email.find(params.require(:id))

email.resend_verification!

email.send_verification!
```

A manager class (there are Email, Sms, and Fax) handles the orchestration. For Email Verification the manager class is `UserCommunicationMethods::Email`.

```ruby
def send_verification!(communication_method)
  UserCommunicationMethods::EmailVerificationWorker.perform_async(communication_method.id)
end

def resend_verification!(communication_method)
    send_verification!(communication_method)
end
```

This class have methods to generate the verification code, send/resend the link, and execute the worker.

The verification code assignation updates the `verification_code` field:
```ruby
def assign_verification_code!(communication_method)
  communication_method.update!(verification_code: SecureRandom.urlsafe_base64(16))
end
```

Finally, the worker `UserCommunicationMethods::EmailVerificationMailer` triggers the mailer class.

In the mailer, the URL is encoded with the UCM id and verification_code:
```ruby
# EmailVerificationMailer
Rails.application.routes.url_helpers.verify_email_url(
    UserCommunicationMethods::Email.encode_url_token(communication_method),

# Email
def encode_url_token(communication_method)
	Base64.urlsafe_encode64(
	  {
		id: communication_method.id,
		code: communication_method.verification_code
	  }.to_json
	)
end
```

Notice this:
```ruby
communication_method.update!(
	verification_sent_at: Time.current,
	verification_code_expires_at: verification_code_valid_hours.hours.from_now,
	verification_attempts: communication_method.verification_attempts + 1
)
```

I can use those fields to do checks for the worker that will send email reminders for those who did not verified.

## Why does the link last 72 hours?

In alpha, it is only valid for 72 hours. In Omega it depends if there's custom setting else is the same amount of hours.

## Manual and automated tests

To verify changes to the landing page at `app/views/user_communication_methods/email_verifications/new.html.haml` I can manually test them by opening the page at `localhost:3000/email_verification/SOMETOKEN`. Comment out the conditionals to see the different paths.

Or automated tests at `spec/mailers/user_communication_methods/email_verification_mailer_spec.rb` for the mailer sent.

# Models

These are the models involved.

## Concerns

- CommunicationMethods
- PortalProviderEntity

Including CommunicationMethods adds:
```ruby
has_many :user_communication_methods, as: :user, dependent: :destroy
```

Including PortalProviderEntity adds:
```ruby
has_one :config, class_name: "PortalConfig", as: :configurable
has_many :portal_email_recipients,
        -> { portal_email_recipient }, as: :parent, class_name: "ShadowUser"
```

## Physician

```ruby
class Physician < ApplicationRecord
  include CommunicationMethods
  include PortalProviderEntity # relation to ShadowUser
end
```

Notice:
- A physician has many `user_communication_methods` because an instance represents an actual physician
- A physician `portal_email_recipient` will also has many `user_communication_methods` because these recipients are individuals.

## ShadowUser

> Users that need to interact with our system but don't have proper accounts. The primary goal here is to be able to verify their communication methods.

```ruby
class ShadowUser < ApplicationRecord
  include CommunicationMethods

  # Parent could be something like a physician / physician group / etc.
  belongs_to :parent, polymorphic: true, optional: true
end
```

Schema:

```ruby
create_table "shadow_users", id: :uuid, default: -> { "uuid_generate_v4()" }, force: :cascade do |t|
    t.string "parent_type"
    t.string "identifier", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.integer "kind", null: false
    t.uuid "parent_id"
    t.index ["parent_type", "parent_id"], name: "index_shadow_users_on_parent_type_and_parent_id"
end
```

## Clinic

```ruby
class Clinic < ApplicationRecord
  include PortalProviderEntity # relation to ShadowUser

  belongs_to :practice
end
```

## Practice

```ruby
class Practice < ApplicationRecord
  include PortalProviderEntity # relation to ShadowUser
  
  has_many :clinics
end
```

## PhysicianGroup

```ruby
class PhysicianGroup < ApplicationRecord
  include PortalProviderEntity # relation to ShadowUser

  # Do not allow deleting physician groups with any associated physicians
  has_many :physicians, dependent: :restrict_with_error
end
```


## UserCommunicationMethod

Model:
```ruby
class UserCommunicationMethod < ApplicationRecord
  belongs_to :user, polymorphic: true
end
```

Schema:
```ruby
create_table "user_communication_methods", id: :uuid, default: -> { "uuid_generate_v4()" }, force: :cascade do |t|
    t.string "user_type", null: false
    t.uuid "user_id", null: false
    t.boolean "primary_method", null: false
    t.integer "verification_status", default: 0, null: false
    t.integer "kind", null: false
    t.string "value", null: false
    t.string "verification_code"
    t.datetime "verification_code_expires_at"
    t.datetime "verification_sent_at"
    t.datetime "verified_at"
    t.integer "verification_attempts", default: 0, null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end
```

# For Milestone 2: Resend from Landing Page ✅

This milestone needs to add two different buttons depending on the outcome of the email verification:

- happy path: link to clinical dashboard
- sad path: link to request verification email

The "Send verification email" button in the edit view of a Physician holds a link like this:

![[200.send.verification.email.png]]

```
https://luxe.alpha.getluna.com/admin/email_verification_links/2cb7f2be-171b-4729-b2f6-14d8b713b4e2
```

Generated by this code:
```ruby
admin_email_verification_link_path(resource.email_address)
```

Also, for the setup recipients there's a link to resend the verification email:

![[201.resend.recipient.png]]

```
https://luxe.alpha.getluna.com/admin/email_verification_links/291f69bb-a3d5-4dbd-ab41-64bec1cbe56d
```

Code that generates this link:
```ruby
admin_email_verification_link_path(recipient.email_address)
```

So, what's the deal with `email_address`?

First, the `resource` and `recipient` instances are instances for Physician and PortalEmailRecipient (ShadowUser).

Both of these classes include the concern `CommunicationMethods` which sets up this association:
```ruby
has_one :email_address, -> { email.primary.order(created_at: :desc) },
        as: :user, class_name: "UserCommunicationMethod"
```

And that's the `email_address`. It's an instance  of `UserCommunicationMethod`.

This is the `resource` instance in the Admin view:
```ruby
#<Physician:0x000000012ca96bb8> {
	   :first_name => "Zayter",
		:last_name => "Testing",
	   :fax_number => "",
:fax_number_validated => false,
	 :phone_number => "",
			:email => "zayter+lun@fo.com",
		  :address => "",
	   :created_at => Sat, 09 May 2020 17:12:59.488796000 PDT -07:00,
	   :updated_at => Mon, 01 Jan 2024 00:24:00.980051000 PST -08:00,
:national_provider_identifier => "0000023449",
			   :id => "ffd12d73-5af8-438f-9815-797575e7ff21",
  :fax_every_visit => false,
	   :hubspot_id => nil,
		   :prefix => "Dr.",
:default_visit_plan => "",
  :care_plan_count => 6,
:care_plans_beginning_treatment_last_90_days_count => 0,
:physician_group_id => nil,
		 :zip_code => nil,
		:region_id => 3,
	   :free_notes => nil,
:is_protocol_enabled => false,
		 :state_id => nil,
		   :npi_id => 5024
}
```

And this is the `email_address` instance:
```ruby
#<UserCommunicationMethod:0x0000000133fd7a08> {
		  :id => "2cb7f2be-171b-4729-b2f6-14d8b713b4e2",
   :user_type => "Physician",
	 :user_id => "ffd12d73-5af8-438f-9815-797575e7ff21",
:primary_method => true,
:verification_status => "unverified",
		:kind => "email",
	   :value => "zayter+lun@fo.com",
:verification_code => nil,
:verification_code_expires_at => nil,
:verification_sent_at => nil,
 :verified_at => nil,
:verification_attempts => 0,
  :created_at => Thu, 24 Feb 2022 11:08:33.343832000 PST -08:00,
  :updated_at => Fri, 02 Aug 2024 14:13:06.182742000 PDT -07:00
}
```

So, in order to add a link to resend the verification email we'd need to do something like this in `EmailVerificationsController`:
```ruby
ucm = UserCommunicationMethod.find(ID)

@resend_url = admin_email_verification_link_path(ucm)
```

However, ==the path will need to be in a new controller because that one needs authentication as a Luxe admin==. **We're going to need to add a new endpoint to trigger this and use a different auth method** because this is a stateless page.

We don't need to do nothing in the controller but in the view in the else case in `app/views/user_communication_methods/email_verifications/new.html.erb`.

# For Milestone 2: Generate portal link after verification ✅

This milestone adds a link to a clinical dashboard after the user successfully verifies their email.

Facts:
- The only classes that include `PortalProviderEntity` are: Clinic, Physician, PhysicianGroup, and Practice
	- These classes can do `send_portal_access_link(provider_email)`
- There's no link between `UserCommunicationMethod` and `Clinic`, `PhysicianGroup`, and `Practice`
	- There's a connection between this class and `Physician`, though
- In the final verification page a UserCommunicationMethod ID is available
	- With this ID we can find the instance and it's user
		- This user could be a ==Physician or a ShadowUser==.

When `ucm.user` is a physician we can send the `send_portal_access_link` method. Else we should send the message to the user's `parent` association.
```ruby
UserCommunicationMethod.find("faaa748c-76ae-4ce1-a954-46275315fdd7").user.respond_to?(:send_portal_access_link)
=> false

UserCommunicationMethod.find("faaa748c-76ae-4ce1-a954-46275315fdd7").user.parent.respond_to?(:send_portal_access_link)
=> true
```

> [!INFO]
> The `parent` method is defined in `ShadowUser` class as described in the ShadowUser section above.

See more details for Ad Hoc link generation at [[Edge to Dashboard Link Generation]]

# For Milestone 3: Automate Email Reminder

This worker should run once a week. Probably early on mondays.

The worker should query `user_communication_methods` records where:

- `kind` is 0 ("email")
- `verified_at` is null and `verification_status` is 0 ("unverified")
- `verification_attempts` < 4

To complete the rule:
> and then every 7 days (once per week) for 3 weeks

I can use the `verification_attempts` field. Anytime an email reminder is sent this number increases. So once it reaches to 4 it means we emailed them four times already.

## Weekly email for three weeks

This will be only sent to recipients that haven't verified their email and haven't opted out of the verification process. This opt-out flag is found in Hubspot via the property `portal_email_cadence`.

This property is a dropdown and has these values:
- Weekly
- Monthly
- Quarterly
- Opt-out

> The internal value of these labels is the same, *downcased*.

> [!Info]
> This value is represented in the database. You can find it by looking at the `portal_email_cadence` field in the `portal_configs` table.
> The value that indicates a use opted-out is `3`.

# Creando UserCommunicationMethod en las Pruebas: ShadowUser o Account?

Resulta que UserCommunicationMethod define una asociación polimorfica con User. Este User puede ser:

- Physician
- ShadowUser
- Account

El modelo Account está reservado para instancias de Patient y Therapist. Para evitar enviar correos a pacientes/therapists, tocaba excluirlos. ¿Cómo los excluyo? Así:
```ruby
recipients = UserCommunicationMethod.where(
	kind: "email",
	verified_at: nil,
	verification_status: "unverified",
	verification_attempts: 0..4,
	user_type: ["Physician", "ShadowUser"]
)
```

Pero en las pruebas toca ser más preciso con las instancias que se crean. En las pruebas ahora necesitaba pasarle como User al ucm un ShadowUser. Tuve que hacer así:
```ruby
physician = create(:physician)
physician_user = create(:shadow_user, :portal_email_recipient, parent: physician)

clinic = create(:clinic)
clinic_user = create(:shadow_user, :portal_email_recipient, parent: clinic)

verified = create(:user_communication_method, :verified, user: physician_user)
unverified = create(:user_communication_method, user: clinic_user)
```

Un ShadowUser tiene un parent que puede ser:

- Practice
- Physician
- PhysicianGroup
- Clinice

# Reintentos de los Workers de Sidekiq

Por defecto, ningún worker se reintenta. Así está configurado en ApplicationWorker:
```ruby
class ApplicationWorker
  include Sidekiq::Worker

  sidekiq_options retry: false
```

Así que no tengo que hacer nada más para el mailer de las 72 horas en caso de que falle pues el usuario recibiría un correo el siguiente Lunes.