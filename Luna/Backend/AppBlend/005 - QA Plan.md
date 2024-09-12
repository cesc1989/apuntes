# Plan de QA para probar todo el sistema

Este plan contempla navegar y usar Luxe, probar integración entre servicios internos, completar ejercicios en la web, disparar background jobs y probar queries de GraphQL.

## Browse and interact in Luxe

- Actions Menu
	- [ ] Browse all sections
	- Assist
		- [ ] Create a deduction
	- Benefit Verifications
		-  [ ]  Filter by any attribute
- Patients / Profile
	- [ ] Search patient
	- [ ] Create care plan for patient
	- [ ] Check Document is attached after completing Intake Form
- Files menu
	- [ ] Open Document shares
		- [ ] Mark one as read
	- [ ] Visit Incoming faxes
- Clinical menu
	- [ ] Go to Auto charts section
		- [ ] View auto chart detail
	- [ ] Go to Plans of Care section
		- [ ] View plan of care detail
- Clinics / Profile
	- [ ] Add new portal recipient
	- [ ] Verify email for Clinical Dashboard
	- [ ] Send Clinical Dashboard email
- Appointments / Detail
	- [ ] Visit appointment detail
- Exercise Programs
	- [ ] View detail of an exercise program
		- [ ] Complete exercises in LE-W. Checklist method
		- [ ] Check recent workout, checklist, shows in the list


## Trigger background jobs manually

Wilted Tree Reminder Worker
- [ ] Trigger worker
- [ ] Check wilted email was received

Clinical Dashboard Send Link Worker

Workers:
```
WiltedTreeExercisesViaWebReminderWorker.perform_async([1])

# Para practice Action On-Demand
ProviderPortalEmailSendingWorker.perform_async("618d2473-9d4c-4db4-9a91-741964a9d791")
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