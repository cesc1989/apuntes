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


### For Milestone 3: Automate Email Reminder

This worker should run once a week.

The worker should query user_communication_methods where:

- `kind` is 0 == "email"
- `verified_at` is null and `verification_status` is 0 ("unverified")
- `verification_attempts` < 4

To complete the rule:
> and then every 7 days (once per week) for 3 weeks

I can use the `verification_attempts` field. Anytime an email reminder is sent this number increases.