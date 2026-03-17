# NEX-3476 - Appointments not syncing to  Candid

Etiquetas: #luna_help_desk 

Reporte:
> Visits in Luxe not going over to Candid.

En la vista XXXX en la columna "Why Unsynced?" aparece el texto "bug?".  El reporte indica que todo appointment con chart firmado debe aparecer en Candid.

## Context

### Signed Visits

En el reporte la persona dice "signed visits". Lo que creo que es appointments completos con chart firmado.

Así lo define Claudio:

> A visit is an  Appointment  with a  Chart . The chart goes through a state machine:
> 
> `pending_for_pdf → pending_for_review → pending_for_approval → signed`

Y así se definió en código:
```ruby
signed_visits = completed_visits.select { |a| a&.chart&.signed? }
```

### Sobre Candid

Claudio dice:
> Candid is the external billing/claims platform. Luna syncs appointment data to Candid as **Encounters** for insurance billing. The link between a synced appointment and Candid is the `CandidEncounter`  model — if a  `CandidEncounter`  record exists for an appointment, it's synced.

El modelo `CandidEncounter` existe en `app/models/candid_encounter.rb`.

### The Sync Flow (Happy Path)

Claudio explica:

```
1. Therapist signs chart
       ↓
2. chart.sign! → sets signature_date, triggers after_update_commit
       ↓
3. Chart#refresh_candid_encounter_via_chart_change!
   (app/models/concerns/candid_encounter_data_modifier.rb ~line 24)
       ↓
4. Checks: appointment.candid_sync_eligible? || appointment.candid_synced?
       ↓ (if eligible)
5. Candid::Egress::SyncAppointmentWorker.perform_async(appointment.id)
       ↓
6. CandidEncounter created/updated → appointment appears as synced in Candid
```

If the chart was not yet eligible at signing time, the appointment is picked up later by the periodic batch job:
```
Candid::Egress::SyncAllEligibleAppointmentsWorker
  → finds all candid_sync_eligible appointments not yet synced
  → enqueues SyncAppointmentWorker for each
```

### ¿Qué hacen los checks?

¿Qué hace `candid_synced?`?
```ruby
def candid_synced?
	candid_encounter.present?
end
```

¿Qué hace `candid_sync_eligible`?

```ruby
def candid_sync_eligible?
	candid_sync_eligible_documented_visit? || candid_sync_eligible_patient_fee_visit?
end
```

Desglosemos los métodos que usa.

```ruby
def candid_sync_eligible_documented_visit?
	return false unless episode.clinic.present?
	return false unless Setting.load_safe("candid_ignored_clinic_keys", []).exclude?(episode.clinic.key)
	return false unless patient.real_patient?
	return false unless therapist.real_therapist?
	return false unless chart_signed?
	return false unless chart.signature_date.present?
	return false unless Appointment
											.candid_sync_eligible_documented_visits_scheduled_date_period.cover?(local_scheduled_date)

	local_signature_date = DateTimeLocalizer.new(chart.signature_date, region).simple_date.to_date
	Appointment.candid_sync_eligible_documented_visits_signature_date_period.cover?(local_signature_date)
end
```

Y luego:
```ruby
# These are all of the patient cancellations we want to sync to Luxe *regardless*
# of their cancellation report status. The Encounter mapper is responsible for indicating Billable Status.
def candid_sync_eligible_patient_fee_visit?
	return false unless patient.real_patient?
	return false unless therapist.real_therapist?
	return false unless patient_fee_state? || (cancelled_by_patient_or_patient_no_show? && waive?)

	Appointment.candid_sync_eligible_patient_fee_visits_scheduled_date_period.cover?(local_scheduled_date.to_date)
end
```

Luego vemos estos:
- `Appointment.candid_sync_eligible_documented_visits_scheduled_date_period`
- `Appointment.candid_sync_eligible_documented_visits_signature_date_period`
- `Appointment.candid_sync_eligible_patient_fee_visits_scheduled_date_period`

Veamos qué son.

```ruby
def self.candid_sync_eligible_documented_visits_scheduled_date_period
	lower_bound = Candid::Client.candid_go_live_date
	upper_bound = nil
	lower_bound..upper_bound
end
```

Donde `candid_go_live_date` es 2024-01-01.

```ruby
def self.candid_sync_eligible_documented_visits_signature_date_period
	# We assess a holding period of N hours after the appointment was SIGNED
	# before syncing to Candid to ensure the appointment state is relatively stable.
	#
	# This holding period exists because we want everything we send to Candid to be final.
	# Appointments can still be unsigned or have their cancellation status change after N
	# hours, but these events should be few and far between.
	lower_bound = Candid::Client.candid_go_live_date
	upper_bound =
		Setting.load_safe("candid_successful_visit_hour_delay", 24)
		.hours
		.before(Time.current)
	lower_bound..upper_bound
end
```


Y
```ruby
def self.candid_sync_eligible_patient_fee_visits_scheduled_date_period
	# We assess a holding period of N business days after the appointment was SCHEDULED
	# before syncing it to Candid to ensure the appointment state is relatively stable.
	#
	# This holding period exists because we want everything we send to Candid to be final.
	# Appointments can still be waived or have their cancellation status change after N
	# business days, but these events should be few and far between.
	#
	# Business days are used here because appointment fee assignment is a Concierge-driven process.
	lower_bound = Candid::Client.candid_go_live_date
	upper_bound =
		[
			candid_sync_eligible_patient_fee_visits_sync_delay_date,
			lower_bound
		].max
	lower_bound..upper_bound
end
```