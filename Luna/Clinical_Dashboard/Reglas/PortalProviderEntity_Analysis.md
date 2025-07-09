# PortalProviderEntity Concern Analysis

## Overview

This document provides a comprehensive analysis of the `PortalProviderEntity` concern and the models that include it, including file locations, line numbers, and code snippets.

## 1. PortalProviderEntity Concern Location

**File**: `app/models/concerns/portal_provider_entity.rb`

## 2. Models Including PortalProviderEntity with File Locations and Code Snippets

### Practice Model

**File**: `app/models/practice.rb:8`
```ruby
class Practice < ApplicationRecord
  include BillingRouteModifier
  include LazyLoadable
  include OutboxIntegration
  include PortalProviderEntity
```

**Required Methods Implementation**:
```ruby
# app/models/practice.rb:174-182
def mailer
  PoweredByPartnerMailer
end

def provider_kind
  return nil unless partner?

  "partner_clinic"
end

# app/models/practice.rb:137-142
def active_care_plan_count
  patient_episodes
    .where(appointments: { scheduled_date: 90.days.ago.. })
    .distinct
    .size
end
```

### Clinic Model

**File**: `app/models/clinic.rb:22`
```ruby
class Clinic < ApplicationRecord
  include BillingRouteModifier
  include LazyLoadable
  include NPIEntity
  include OutboxIntegration
  include PortalProviderEntity
```

**Required Methods Implementation**:
```ruby
# app/models/clinic.rb:174-184
def provider_kind
  "clinic"
end

def mailer
  ClinicMailer
end

def active_care_plan_count
  Episode.where(clinic: self).count
end
```

### Physician Model

**File**: `app/models/physician.rb:12`
```ruby
class Physician < ApplicationRecord
  include CommunicationMethods
  include Contactable
  include DocumentSubjectable
  include NPIEntity
  include OutboxIntegration
  include PortalProviderEntity
  include RegionTenant
```

**Required Methods Implementation**:
```ruby
# app/models/physician.rb:128-138
def mailer
  PhysicianMailer
end

def provider_kind
  "physician"
end

def active_care_plan_count
  care_plans_with_recently_completed_appointments.size
end
```

### PhysicianGroup Model

**File**: `app/models/physician_group.rb:8`
```ruby
class PhysicianGroup < ApplicationRecord
  include DocumentSubjectable
  include OutboxIntegration
  include PortalProviderEntity
```

**Required Methods Implementation**:
```ruby
# app/models/physician_group.rb:16-34
def mailer
  PhysicianGroupMailer
end

def provider_kind
  "physician_group"
end

def active_care_plan_count
  physicians.collect(&:care_plans_with_recently_completed_appointments).flatten.size
end
```

## 3. Global Configuration Settings

### Provider Portal Emails Enabled Setting

**File**: `spec/factories/settings.rb:306-310`
```ruby
trait :provider_portal_emails_enabled do
  key { "provider_portal_emails_enabled" }
  value { "true" }
  setting_type { "boolean" }
end
```

ActiveRecord query:
```ruby
Setting.find_by(key: "provider_portal_emails_enabled")
```

## 4. Key Features of PortalProviderEntity Concern

### Associations

- `has_one :config, class_name: "PortalConfig", as: :configurable`
- `has_many :portal_email_recipients` (ShadowUser with `portal_email_recipient` scope)

### Permission Delegates

- `can_sign_patient_pocs` and `can_sign_patient_pocs?`
- `can_view_patient_charts` and `can_view_patient_charts?`
- `can_view_assigned_physicians` and `can_view_assigned_physicians?`

### Communication Settings

- `portal_email_cadence` (weekly, monthly, quarterly, never)
- `signature_strategy_rule` (link_enrollment, fax_override)

### Referral Settings

- `disable_tip_prompt_for_referrals` and `disable_tip_prompt_for_referrals?`

## 5. Key Rules for Sending Provider Portal Links

### Prerequisites for Portal Activation

1. **Email cadence** must be set (cannot be blank or "never") - configured via `PortalConfig`
2. **Email recipients** must exist via `portal_email_recipients` association
3. **Active care plan count** must meet or exceed `config.active_case_threshold`
4. **Global setting** `provider_portal_emails_enabled` must be true

### Portal Link Sending Rules

1. **Portal must be active** (`portal_active?` returns true)
2. **Recipients must be verified** or have verification disabled
3. **Rate limiting**: Cannot send within 10 minutes of previous send (enforced by `PortalEmailDuplicateAttempt`)
4. **Required methods**: Each model must implement:
   - `mailer` - Returns appropriate mailer class
   - `provider_kind` - Returns provider type string  
   - `active_care_plan_count` - Returns count of active care plans

### Email Verification Categories

- **Verified emails**: `verified_portal_email_recipient_emails`
- **Verification disabled**: `verification_disabled_portal_email_recipient_emails`  
- **Unverified emails**: `unverified_portal_email_recipient_emails` (excluded from sends)

### Exception Handling

- `PortalEmailUpstreamError` - Portal URL generation fails
- `PortalEmailNoneVerified` - No verified recipients exist  
- `PortalEmailDuplicateAttempt` - Send attempted within 10 minutes

## 6. Configuration Architecture

### PortalConfig Association

Each model has:
```ruby
# From PortalProviderEntity concern
has_one :config, class_name: "PortalConfig", as: :configurable
```

### Portal Email Recipients Association

```ruby
# From PortalProviderEntity concern
has_many :portal_email_recipients,
         -> { portal_email_recipient }, as: :parent, class_name: "ShadowUser"
```

## 7. Main Portal Methods Implementation

### Portal Active Check

```ruby
# From PortalProviderEntity concern
def portal_active?
  return false if portal_email_cadence.blank? || portal_email_cadence == "never"
  return false if portal_email_recipients.empty?

  active_care_plan_count >= config.active_case_threshold
end
```

### Send Portal Access Link (Single Recipient)

Request a new Provider Portal link from the Expired Link page.

![[request new cd link.png]]

Clicking the button makes a request to Clinical Dashboard `POST /refresh` endpoint which does a request to Edge `POST /api/v1/internal/provider_communications/send_portal_access_link` endpoint.

```ruby
def send_portal_access_link(provider_email)
  # Uses ProviderPortalService to generate portal URLs
  # Sends email using the model's mailer
  # Records transmission and updates link_last_sent_at
  # Creates HubSpot engagement note if configured
end
```

### Send Portal (All Recipients)

Send provider portal links to all providers in their corresponding cadency.

`ProviderPortalEmailsSendingWorker` runs weekly at 8am. For each found `PortalConfig` it runs  `ProviderPortalEmailSendingWorker`. This last worker is where the `send_portal` method is used.

```ruby
def send_portal(force: false)
  # Checks if portal is active (unless force: true)
  # Prevents duplicate sends within 10 minutes
  # Requires verified or verification-disabled email recipients
  # Generates portal URLs for all valid recipients
  # Sends individual emails and tracks success/failure
  # Updates link_last_sent_at timestamp
  # Creates HubSpot engagement notes with results
end
```

It's also used in providers profiles in Luxe. There's a button to send the provider portal link on demand.

## 8. Related Services and Workers

### Services

- **`ProviderPortalService`** (`app/services/provider_portal_service.rb`) - Interfaces with Partner Portal API to generate portal URLs

### Workers

- **`ProviderPortalEmailSendingWorker`** (`app/workers/provider_portal_email_sending_worker.rb`) - Queues portal email sending for individual providers

### Controllers

- **`Api::V1::Internal::ProviderCommunicationsController`** - Handles portal access link requests
- **`ClinicalDashboard::Api::V1::SessionsController`** - Manages dashboard sessions and refresh requests

## 9. Configuration Model

- **`PortalConfig`** (`app/models/portal_config.rb`) - Stores portal configuration settings including email cadence, permissions, and active case thresholds

## Summary

The `PortalProviderEntity` concern provides a unified interface for all provider types (physicians, clinics, practices, physician groups) to access the provider portal system with consistent permission management, email sending, and configuration handling through the polymorphic `PortalConfig` association. Each model must implement the required methods (`mailer`, `provider_kind`, `active_care_plan_count`) to fulfill the interface contract.