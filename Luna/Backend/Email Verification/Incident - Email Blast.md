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

# Debugging Queries

> [!Important]
> Below is the snippet provided that would select elegible physicians for the Dashboard

```sql
LEFT JOIN (
  SELECT
    id AS physician_id
    ,COUNT(episodes.id)
  FROM physicians
  LEFT JOIN episodes ON episodes.physician_id = physicians.id
  LEFT JOIN appointments ON appointments.episode_id = episodes.id
  LEFT JOIN regions ON regions.id = appointments.region_id
  LEFT JOIN portal_configs ON portal_configs.configurable_id = physicians.id
  WHERE appointments.state = 2
    AND appointments.scheduled_date >= NOW() - INTERVAL '90 days'
  GROUP BY appointments.episode_id
  HAVING COUNT(appointments.id) > config.active_case_threshold
) AS eligible_physicians ON eligible_physicians.physician_id = physicians.id
```

This would be the Active Record representation of the above query:
```ruby
Episode
  .joins(:appointments)
  .merge(Appointment.completed)
  .where(appointments: { scheduled_date: 90.days.ago.. })
  F.distinct(:episode_id)
```

Next, all the queries ran to debug this case.

## Most Recent Debug query using Subqueries

```sql
select ucm.user_type as ucm_type,
  ucm.user_id as ucm_id,
  ucm.value as ucm_email,
  (case ucm.verification_status
    when 0 then 'unverified'
    when 5 then 'verified'
    when 10 then 'verification_disabled'
    end
  ) as verification_status,
  ucm.verification_attempts,
  ucm.verification_sent_at,
  su.parent_type as recipient_parent_type,
  su.identifier as recipient_email,
  (case su.kind
    when 0 then 'portal_email_recipient'
    when 1 then 'escalation_email_recipient'
    end
  ) as recipient_kind,
  (case pc.portal_email_cadence
    when 0 then 'weekly'
    when 1 then 'monthly'
    when 2 then 'quarterly'
    when 3 then 'never'
    end
  ) as portal_email_cadence,
  pc.configurable_type as provider_kind,
  (case pc.configurable_type
    when 'Physician' then (select concat(first_name, ' ', last_name) from physicians where id = pc.configurable_id)
    when 'PhysicianGroup' then (select name from physician_groups where id = pc.configurable_id)
    when 'Clinic' then (select provider_name from clinics where id = pc.configurable_id)
    when 'Practice' then (select name from practices where id = pc.configurable_id)
    end
  ) as provider_name
from user_communication_methods ucm
join (
  select distinct on (identifier) * 
  from shadow_users
) as su on ucm.user_id = su.parent_id
join portal_configs pc on ucm.user_id = pc.configurable_id
where ucm.kind = 0
and ucm.user_type in ('Physician', 'ShadowUser')
and ucm.verified_at is null
and ucm.verification_attempts between 0 and 3
and pc.portal_email_cadence != 3
;
```

## Same query but with CTE for better readability

```sql
/*
 * Aquí voy a escribir la query para Weekly Email Verification en modo CTE
*/
with communication_method as (
  select
    ucm.user_type,
	  ucm.user_id,
	  ucm.value as user_email,
		(case ucm.verification_status
		  when 0 then 'unverified'
		  when 5 then 'verified'
		  when 10 then 'verification_disabled'
		  end
		) as verification_status,
		ucm.verification_attempts,
		ucm.verification_sent_at
  from user_communication_methods ucm
  where ucm.kind = 0
	and ucm.user_type in ('Physician', 'ShadowUser')
	and ucm.verified_at is null
	and ucm.verification_attempts between 0 and 3
),
shadow_user as (
  select distinct on (identifier)
    su.parent_id,
    su.parent_type as recipient_parent_type,
    su.identifier as recipient_email,
    (case su.kind
      when 0 then 'portal_email_recipient'
      when 1 then 'escalation_email_recipient'
      end
    ) as recipient_kind
  from shadow_users su
),
portal_configuration as (
  select
    pc.configurable_id,
	  (case pc.portal_email_cadence
	    when 0 then 'weekly'
	    when 1 then 'monthly'
	    when 2 then 'quarterly'
	    when 3 then 'never'
	    end
	  ) as portal_email_cadence,
	  pc.configurable_type as provider_kind,
	  (case pc.configurable_type
	    when 'Physician' then (select concat(first_name, ' ', last_name) from physicians where id = pc.configurable_id)
	    when 'PhysicianGroup' then (select name from physician_groups where id = pc.configurable_id)
	    when 'Clinic' then (select provider_name from clinics where id = pc.configurable_id)
	    when 'Practice' then (select name from practices where id = pc.configurable_id)
	    end
	  ) as provider_name
  from portal_configs pc
  where pc.portal_email_cadence != 3
),
elegible_physician as (
	SELECT
	  phy.id AS physician_id,
	  phy.first_name,
	  phy.last_name,
	  COUNT(episodes.id)
	FROM physicians phy
	LEFT JOIN episodes ON episodes.physician_id = phy.id
	LEFT JOIN appointments ON appointments.episode_id = episodes.id
	LEFT JOIN regions ON regions.id = appointments.region_id
	LEFT JOIN portal_configs ON portal_configs.configurable_id = phy.id
	WHERE appointments.state = 2 /* completed */
	AND appointments.scheduled_date >= NOW() - INTERVAL '90 days' /* desde hace tres meses */
	GROUP BY appointments.episode_id, phy.id, portal_configs.active_case_threshold
	HAVING COUNT(appointments.id) > portal_configs.active_case_threshold /* para esto es el count en el select. tiene que ser mayor al treshold */
),
weekly_verifiable as (
	select
	  cm.user_type,
	  cm.user_id,
	  cm.user_email,
	  cm.verification_status,
	  cm.verification_attempts,
	  cm.verification_sent_at,
	  su.parent_id,
	  su.recipient_parent_type,
	  su.recipient_email,
	  su.recipient_kind,
	  pc.configurable_id,
	  pc.portal_email_cadence,
	  pc.provider_kind,
	  pc.provider_name,
	  ep.physician_id,
	  ep.first_name,
	  ep.last_name
	from communication_method cm
	join shadow_user su on su.parent_id = cm.user_id
	join portal_configuration pc on pc.configurable_id = cm.user_id
	left join elegible_physician ep on ep.physician_id = cm.user_id
)
select *
from weekly_verifiable
;
```