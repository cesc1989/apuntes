# Incident - Email Blast

The query I built to grab user communication methods to email to caused trouble. More than 10% of emails bounced.

Here are some notes on the query and the data model behind.

# Findings

## UCM user_type

In the query I did:

```ruby
user_type: ["Physician", "ShadowUser"]
```

**but ShadowUser already includes Physician**.

Qs:
- Is there a difference?
- Does this cause something different?

## ShadowUser

Definition:
> Users that need to interact with our system but don't have proper accounts. The primary goal here is to be able to verify their communication methods.

ShadowUser is only for:

- Practice
- PhysicianGroup
- Physician*
- Clinic

This model defines an enum `kind`:
```ruby
  enum kind: {
    portal_email_recipient: 0,
    escalation_email_recipient: 1
  }
```

And it includes the concern `CommunicationMethods` so it has these associations:
```ruby
has_many :user_communication_methods, as: :user, dependent: :destroy

has_one :email_address, -> { email.primary.order(created_at: :desc) },
            as: :user, class_name: "UserCommunicationMethod"

has_one :email_address_pending_verification, -> { email.unverified.order(created_at: :desc) },
            as: :user, class_name: "UserCommunicationMethod"
```

## Physician is ShadowUser Twice

The Physician model does this:
```ruby
class Physician
  include PortalProviderEntity

  has_many :escalation_email_recipients,
    -> { escalation_email_recipient }, as: :parent, class_name: "ShadowUser"
end
```

And the concern `PortalProviderEntity` adds this to the model:
```ruby
has_many :portal_email_recipients,
	 -> { portal_email_recipient }, as: :parent, class_name: "ShadowUser"
```

This is why a Physician is twice a ShadowUser.