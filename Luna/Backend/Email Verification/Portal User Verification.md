# User Email Verification

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

## Flow for sending / resending link

A UserCommunicationMethod instance receives the `send_verification!` or the  `resend_verification!` messages

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

Finally, the worker `UserCommunicationMethods::EmailVerificationMailer` triggers the mailer class.

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

To verify changes to the landing page at `app/views/user_communication_methods/email_verifications/new.html.haml` I can manually test them by opening the page at `localhost:3000/email_verification/SOMETKKEN`. Comment out the conditionals to see the different paths.

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

    t.index ["user_type", "user_id", "kind", "value"], name: "idx_user_communication_methods_kind_val_uniq", unique: true
    t.index ["user_type", "user_id", "kind"], name: "idx_user_communication_methods_primary_kind_uniq", unique: true, where: "(primary_method = true)"
  end
```


# For Milestone 3: Automate Email Reminder

This worker should run once a week. Probably early on mondays.

The worker should query `user_communication_methods` records where:

- `kind` is 0 ("email")
- `verified_at` is null and `verification_status` is 0 ("unverified")
- `verification_attempts` < 4

To complete the rule:
> and then every 7 days (once per week) for 3 weeks

I can use the `verification_attempts` field. Anytime an email reminder is sent this number increases. So once it reaches to 4 it means we emailed them four times already.

# For Milestone 2: Resend from Landing Page

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