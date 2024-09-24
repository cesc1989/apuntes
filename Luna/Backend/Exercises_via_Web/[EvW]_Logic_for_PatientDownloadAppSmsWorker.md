# [EvW] Logic for PatientDownloadAppSmsWorker

So I was about to do something like this:
```ruby
DownloadAppSmsWorker.perform_at(new_time, patient_id, (index + 1), id)
```

However I see this in `patient.rb`:
```ruby
def clear_sms_download_app_reminders
	SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "1st")
	SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "2nd")
	SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "3rd")
end
```

The `clear_sms_download_app_reminders` is used to clear the download app worker after a patient logs in.

`PatientDownloadAppSmsWorker` being used in Patient but originally scheduled in ExerciseProgram is what made me do signature

```ruby
def perform(patient_id, kind, exercise_program_id = nil)
```

So I think we should keep only one download app worker and instead do this:

**Fetch the most recent exercise program in** `**clear_sms_download_app_reminders**`

So the method would be:
```ruby
def clear_sms_download_app_reminders
	ep = most_recent_exercise_program # this imethod exists already

	SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "1st", ep.id)
	SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "2nd", ep.id)
	SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "3rd", ep.id)
end
```

I'll have to update some tests but in return I would be able to:

- remove `exercise_program_id = nil` from the worker signature.
- remove the conditional in `ExerciseProgram.find if exercise_program_id.present?`
- and remove `return "" if exercise_program.blank?` in ExerciseProgramLinkHelper

## Scheduling and Descheduling

I was worried this might raise exception because of a nil exercise_program_id
```ruby
class PatientDownloadAppSmsWorker < ApplicationWorker
	def perform(patient_id, kind, exercise_program_id)
		patient = Patient.find(patient_id)
		exercise_program = ExerciseProgram.find(exercise_program_id)
		# (...)
	end
end
```

However, the logic above I consider it valid because the scheduling only happens in ExerciseProgram model. The descheduling only happens in Patient model and it needs the exercise_program_id because the SidekiqManager needs all the arguments.

Other than that, `PatientDownloadAppSmsWorker` isn’t used anywhere else as we can see after making a text search:
```bash
$ ag "PatientDownloadAppSmsWorker" ./app/
app/models/exercise_program.rb
144:      PatientDownloadAppSmsWorker.perform_at(new_time, patient_id, (index + 1).ordinalize, id)

app/models/patient.rb
848:    SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "1st", exercise_program&.id)
849:    SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "2nd", exercise_program&.id)
850:    SidekiqManager.dequeue_scheduled(PatientDownloadAppSmsWorker, id, "3rd", exercise_program&.id)

app/workers/patient_download_app_sms_worker.rb
4:class PatientDownloadAppSmsWorker < ApplicationWorker
```