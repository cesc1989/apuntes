# What is involved to create Dashboard links?

Links are generated from a request in Luxe to Clinical Dashboard.

## Files

These are all the files where things happen:

- PortalProviderEntity
	- methods: `send_portal_access_link`, `send_portal`
- Api::V1::Internal::ProviderCommunicationsController
- ProviderPortalEmailSendingWorker
- ProviderPortalEmailsSendingWorker
- ProviderPortalService
- Mailers:
	- PoweredByPartnerMailer
	- ClinicMailer
	- PhysicianMailer
	- PhysicianGroupMailer
	- Each mailer implements:
		- refresh_portal_access_link (used for the ad-hoc version)
		- portal (used by the workers)

There are two paths links are created:

1. Background job
2. Requesting link manually

## Background Jobs

Weekly on Mondays, `ProviderPortalEmailsSendingWorker` , to the correct subset of providers, triggers a `ProviderPortalEmailSendingWorker` execution.

In the singular worker, the `#send_portal` method is the one in used.

## Controller for Ad-Hoc link generation

In `Api::V1::Internal::ProviderCommunicationsController`, the `send_portal_access_link` action, after validations, executes:
```ruby
provider.send_portal_access_link(provider_email)
```

## Request link generation to Clinical Dashboard

The communication between services happens in `ProviderPortalService`. This is the place where the payload is generated and the request made and handled:
```ruby
def recipient_params_for(entity, recipient_emails)
    recipient_emails.map do |email|
      {
        provider_name: entity.name,
        provider_kind: entity.provider_kind,
        provider_id: entity.id,
        portal_recipient_email: email
      }
    end
  end
```