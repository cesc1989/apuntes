# Plan de QA para probar el sistema

Este plan contempla navegar y usar Luxe, probar integración entre servicios internos, completar ejercicios en la web, disparar background jobs y probar queries de GraphQL.

## Browse and interact in Luxe

- Actions Menu
	- [x] Browse all sections
	- Assist
		- [x] Create a deduction
	- Benefit Verifications
		-  [x]  Filter by any attribute
- Patients / Profile
	- [x] Search patient
	- [ ] Create care plan for patient
	- [ ] Check Document is attached after completing Intake Form
- Files menu
	- [x] Open Document shares
		- [x] Mark one as read
	- [x] Visit Incoming faxes
- Clinical menu
	- [x] Go to Auto charts section
		- [x] View auto chart detail
	- [x] Go to Plans of Care section
		- [ ] View plan of care detail
- Clinics / Profile
	- [ ] Add new portal recipient
	- [ ] Verify email for Clinical Dashboard
	- [ ] Send Clinical Dashboard email
- Appointments / Detail
	- [ ] Visit appointment detail
- Exercise Programs
	- [x] View detail of an exercise program
		- [ ] Complete exercises in LE-W. Checklist method
		- [ ] Check recent workout, checklist, shows in the list


## Trigger background jobs manually

Wilted Tree Reminder Worker
- [ ] Trigger worker
- [ ] Check wilted email was received

```
WiltedTreeExercisesViaWebReminderWorker.perform_async([1])
```

Clinical Dashboard Send Link Worker

Workers:
```
# Para practice Action On-Demand
ProviderPortalEmailSendingWorker.perform_async("618d2473-9d4c-4db4-9a91-741964a9d791")
```


Athena Clinical Dashboard data workers
- [ ] Athena Provider Portal Physician Group
- [ ] Athena Provider Portal Practices

```
Athena::ProviderPortalPhysicianGroupWriterWorker.perform_async
Athena::ProviderPortalPartnerClinicWriterWorker.perform_async
```

## Test GraphQL queries

¿Cómo hago esto?



## Misc Tests

acts-as-taggable-on 9.0.0

```ruby
e1 = Exercise.first

e1.body_part_list.add("Guapo")
e1.save

ap e1.body_part_list

e1.body_part_list.remove("Guapo")
e1.save

ap e1.body_part_list
```

paranoia 2.5.0

```ruby
i1 = Insurance.first
i1.deleted_at

i1.destroy

Insurance.with_deleted
Insurance.with_deleted.count

Insurance.only_deleted.count
```

stateful_enum 0.7.0

```ruby
Appointment.stateful_enum
app1 = Appointment.first

app1.stateful_enum


[11] pry(main)> app1.state
=> "no_show"
[12] pry(main)> app1.can_finish?
=> false
[13] pry(main)> app1.uncancel
=> false
[14] pry(main)> app1.uncancel!
RuntimeError: Invalid transition


app2 = Appointment.order("RANDOM()").first
app2.uncancel
```

audited 5.2.0

Therapist
```ruby
t1 = Therapist.order("RANDOM()").first
t1.audits
t1.audits.first.audited_changes

t1.update(accepting_new_patients: "open_to_new_patients")

t1.accepting_new_patients_changed_at
```

Appointment
```ruby
a1 = Appointment.order("RANDOM()").first
a1.audits
a1.audits.first.audited_changes

a1.rating_altered?
a1.update(rate_value: 4)
a1.rating_altered?
```