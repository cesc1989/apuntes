# [EvW] Discovery

# Questions
- What is the flow to load exercises for a patient?
    - Initial API calls from Mobile App
    - Nothing specific apparently
    - How do they get patient/Therapist ID?
        - Because patients/therapists install the app
- What is the alpha domain to make requests to?
    - `https://api2.alpha.getluna.com`
- Do requests need to be authenticated?
    - Yes. Auth is covered by [gem devise_token_auth](https://devise-token-auth.gitbook.io/devise-token-auth/usage/model_concerns).
    - Use Account model
        - Account model has Patient
# Findings

*NOTE: Ryan said not to reuse any of these endpoints as they’re for the mobile app.*

## Routes

```ruby
namespace :api do
	namespace :therapist do
		resources :exercise_tags, only: %i[index] do
			collection { get :counts }
		end
		resources :exercises, only: %i[index show]
		resources :exercise_programs
		resources :workouts, only: %i[index show]
	end

	namespace :patient do
		resources :exercises, only: %i[index show]
		resources :exercise_programs, only: %i[index show]
		resources :workouts do
			member { post :times }
		end
	end
end
```


## Concerns and Controllers

**Api::V3::Concerns::ExerciseReadable**
Provides listing and filtering exercises.

This concern is included in:

- *Api::V3::Therapist::ExercisesController*
- *Api::V3::Patient::ExercisesController*

**Api::V3::Concerns::ExerciseProgramReadable**
This concern is included in:

- *Api::V3::Therapist::ExerciseProgramsController*: API for therapists to CRUD their exercise programs
- *Api::V3::Patient::ExerciseProgramsController:* API for patients to read their exercise programs

**Api::V3::Concerns::WorkoutReadable**
This concern is included in:

- *Api::V3::Patient::WorkoutsController*: API for patient app to upload workout data
- *Api::V3::Therapist::WorkoutsController*: Readonly API for workouts


## Models
- Exercise
- BaseExercise
    - Hold base data for exercise routines: instructions, vídeo url, etc
- ExerciseProgram
- Workout
    - Hold data of exercises performed by patients.

