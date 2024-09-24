# How is a workout completion_percent calculated?

## Workout Schema

![[workout.table.png]]

The important field here is `exercise_times`. It is a hash that collects details about each exercise the patient performs.

Another important bit is these associations:

- `workout` belongs to `exercise_program`
- `exercise_program` has many `exercise_assignments`

## Operation

This is the operation to calculate the `completion_percent`:
```ruby
workout.completion_percent = workout.exercise_times.size.to_f / workout.exercise_program.exercise_assignments.size
```

Where:
```ruby
workout.exercise_times.size
```

are the exercises the patient performed.

And:
```ruby
workout.exercise_program.exercise_assignments.size
```

are the actual exercises assigned by the therapist when creating the Exercise Program for the patient.

## Video Workout

This workout was completed via the standard way (patient watched all videos):
```ruby
 id: "00012ea0-c42f-4a2c-87e2-957b27a7efa3",
 started_at: Fri, 15 Dec 2023 15:04:51.596000000 PST -08:00,
 ended_at: nil,
 completion_percent: 0.8571428571428571,
 pain_level: 0,
 exercise_program_id: "4c48e499-3604-4012-ba58-05e46c9ddaa2",
 created_at: Fri, 15 Dec 2023 15:04:51.874226000 PST -08:00,
 updated_at: Fri, 15 Dec 2023 15:06:28.059385000 PST -08:00,
 exercise_times:
  {"0c1446ec-9c67-48a1-b835-d4b49e7ea4de"=>
    {"times"=>[{"end_time"=>"2023-12-15T15:06:27.581-08:00", "start_time"=>"2023-12-15T15:06:27.226-08:00"}], "assignment"=>{"hold"=>0, "reps"=>3, "sets"=>1}, "actual_seconds"=>0.355, "expected_seconds"=>30},
   "14799e1e-cecd-4b7b-9a69-a229d5acb183"=>
    {"times"=>[{"end_time"=>"2023-12-15T15:04:55.413-08:00", "start_time"=>"2023-12-15T15:04:54.696-08:00"}], "assignment"=>{"hold"=>0, "reps"=>1, "sets"=>1}, "actual_seconds"=>0.717, "expected_seconds"=>30},
   "1cb09734-8f1c-4586-b091-c03b05c937d5"=>
    {"times"=>[{"end_time"=>"2023-12-15T15:04:52.987-08:00", "start_time"=>"2023-12-15T15:04:51.811-08:00"}], "assignment"=>{"hold"=>5, "reps"=>2, "sets"=>3}, "actual_seconds"=>1.176, "expected_seconds"=>90},
   "5038751c-7eff-4559-885d-9ac3f8b6b60c"=>
    {"times"=>[{"end_time"=>"2023-12-15T15:06:26.024-08:00", "start_time"=>"2023-12-15T15:04:55.415-08:00"}],
     "assignment"=>{"hold"=>3, "reps"=>2, "sets"=>2},
     "actual_seconds"=>90.609,
     "expected_seconds"=>60},
   "d1dbcbd8-04a0-4a51-9ac4-1be4f6edb4d2"=>
    {"times"=>[{"end_time"=>"2023-12-15T15:06:27.225-08:00", "start_time"=>"2023-12-15T15:06:26.900-08:00"}], "assignment"=>{"hold"=>3, "reps"=>1, "sets"=>1}, "actual_seconds"=>0.325, "expected_seconds"=>30},
   "d2167a65-0296-441c-be97-3d46487b5e61"=>
    {"times"=>[{"end_time"=>"2023-12-15T15:06:26.897-08:00", "start_time"=>"2023-12-15T15:06:26.567-08:00"}], "assignment"=>{"hold"=>0, "reps"=>1, "sets"=>1}, "actual_seconds"=>0.33, "expected_seconds"=>30}},
 format_type: "default">
```

This workout has 6 completed exercises:
```ruby
Workout.first.exercise_times.size
=> 6
```

And the EP has 7 assigned exercises:
```ruby
Workout.first.exercise_program.exercise_assignments.size
=> 7
```

Applied in the formula:
```ruby
workout.completion_percent = 6.to_f / 7
=> 0.8571428571428571
```

## Checklist Workout

```ruby
id: "33c4f653-ec4e-476b-981c-513ca36d3a17",
 started_at: Tue, 30 Jul 2024 14:43:16.947000000 PDT -07:00,
 ended_at: Tue, 30 Jul 2024 14:43:21.775000000 PDT -07:00,
 completion_percent: 0.5,
 pain_level: 1,
 exercise_program_id: "9852e0cd-1a28-4576-9d2d-2a609e86946a",
 created_at: Tue, 30 Jul 2024 14:43:17.529417000 PDT -07:00,
 updated_at: Tue, 30 Jul 2024 14:43:22.049873000 PDT -07:00,
 exercise_times:
  {"9c802ca4-61a9-44de-bb55-a7107da906d4"=>
    {"times"=>[{"end_time"=>"2024-07-30T14:43:19.571-07:00", "start_time"=>"2024-07-30T14:43:18.571-07:00"}], "assignment"=>{"hold"=>3, "reps"=>2, "sets"=>3}, "actual_seconds"=>1.0, "expected_seconds"=>90}},
 format_type: "web_checklist">
```

This workout has one completed exercises:
```ruby
Workout.find("33c4f653-ec4e-476b-981c-513ca36d3a17").exercise_times.size
=> 1
```

And its EP has X assigned exercises:
```ruby
Workout.find("33c4f653-ec4e-476b-981c-513ca36d3a17").exercise_program.exercise_assignments.size
=> 2
```

Applied in the formula:
```ruby
workout.completion_percent = 1.to_f / 2
=> 0.5
```