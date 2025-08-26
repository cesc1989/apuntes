# Medicare Threshold HubSpot Sync Triggers

Created by Claude.

Where is `KXModifier::MedicareDollarThresholdStatusRefresherService` triggered?

## Overview

This document outlines what triggers the Medicare threshold status refresh and subsequent HubSpot property updates.

## Trigger Flow

```
Appointment/Episode changes → after_update_commit hook → 
maybe_refresh_medicare_dollar_threshold_statuses → 
KXModifier::MedicareDollarThresholdStatusRefresherWorker.perform_async → 
KXModifier::MedicareDollarThresholdStatusRefresherService.refresh → 
update_medicare_threshold_in_hubspot → 
Hubspot::UpdateContactPropertiesWorker (updates medicare_threshold_exceeded property)
```

## Specific Triggers

### 1. Appointment Model (`app/models/appointment.rb`)

**Hook**: `after_update_commit :maybe_refresh_medicare_dollar_threshold_statuses`

**Triggers when any of these appointment fields change**:
- `scheduled_date_previously_changed?` - Appointment date changes
  - *Impact*: Could affect which year's threshold applies
- `state_previously_changed?` - Appointment status changes 
  - *Examples*: started, completed, cancelled, no_show, etc.
  - *Impact*: Affects billing calculations and threshold accumulation
- `episode_previously_changed?` - Appointment moved to different care plan
  - *Impact*: Could move appointment between Medicare and non-Medicare plans

### 2. Episode Model (`app/models/episode.rb`)

**Hook**: `after_update_commit :maybe_refresh_medicare_dollar_threshold_statuses`

**Triggers when**:
- `insurance_previously_changed?` - Care plan insurance changes
  - *Examples*: Switching from commercial to Medicare, Medicare to Medicare Advantage, etc.
  - *Impact*: Determines whether patient has Medicare threshold tracking

## HubSpot Property Updates

The service updates the `medicare_threshold_exceeded` property with:
- **"Yes"** - Patient has exceeded Medicare threshold for current year
- **"No"** - Patient has Medicare plans but hasn't exceeded threshold for current year
- **No update** - Patient doesn't have Medicare plans or no threshold data exists

## Key Benefits

- **Real-time updates**: HubSpot property stays current with any Medicare-relevant changes
- **Year-over-year accuracy**: Prevents stale "Yes" values from previous years
- **Automatic cleanup**: Handles insurance plan changes properly
- **Responsive to billing events**: Updates when appointments complete/cancel

## Implementation Files

- `app/models/appointment.rb` - Appointment change triggers
- `app/models/episode.rb` - Episode/care plan change triggers  
- `app/workers/kx_modifier/medicare_dollar_threshold_status_refresher_worker.rb` - Background job
- `app/services/kx_modifier/medicare_dollar_threshold_status_refresher_service.rb` - Core logic + HubSpot sync