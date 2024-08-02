# How is a workout completion_percent calculated?

## Workout Schema

![[Pasted image 20240802153201.png]]

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

## Examples

