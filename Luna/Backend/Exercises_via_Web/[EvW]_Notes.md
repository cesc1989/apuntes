# [EvW] Notes

## Algunas notas

- `ExerciseProgram` is the core of this app
    - It makes the relationship between a `Patient` and `Exercises`
- `ExerciseProgram` association in `Patient` model uses the `checked_out` scope
    - This scope is defined in the `ExerciseProgram` model. It queries records where `checked_out: true`
- In the current setup, to make requests an authenticated account is needed.
    - Code that runs auth `before_action :authenticate_account!` in `ApiController`
- La asociación entre `Patient` y `Therapist` con `Account` se expresa en el concern `Profile`
    - El concern `Profile`  está en `app/models/concerns/profile.rb`


# Flujo para guardar los avances del workout

**1) User presses “Start workout” button.**

Make a request to create a workout for the `exercise_program`.

    POST /workouts
    {
      "exercise_program_id": ID,
      "started_at": datetime
    }

**2) User transitions from one exercise to another.**

Make request to save elapsed time between exercises to be able to calculate their progress.

    POST /workouts/[ID]/times
    {
      "exercise_id": ID,
      "start_time": datetime,
      "end_time": datetime,
      "account_id": UUID
    }

**3) User ends the workout and chooses a pain score.**

Make request to update the workout created in the initial step.

    PUT /workouts/[ID]
    {
      "account_id": patient_account_id,
      "pain_level": integer,
      "ended_at": datetime
    }

Comentario de Ryan:

> I think mobile only sends `ended_at` when the user submits a pain score, but I would think `ended_at` should be sent immediately after transitioning away from the final exercise.
> 
> In which case you would call `PATCH` twice instead of once.


# Sobre la gema Devise Token Auth

Para autenticación de peticiones usan la gema [Devise Token Auth](https://devise-token-auth.gitbook.io/devise-token-auth/usage/routes). Está configurada solo para el modelo `Account`.

    include DeviseTokenAuth::Concerns::User

Veo que la usan solo para la V2 del API pero esto es solo para que haya rutas de registro y/o sesiones.

    namespace :v2 do
      namespace :patient do
        mount_devise_token_auth_for "Account"
      end
    
      namespace :therapist do
        mount_devise_token_auth_for "Account"
      end
    end

Veo que en `ApiPatientController` se usa el método `current_account` pero cuando hago una búsqueda general no encuentro su definición. Eso significa que está definido por la gema. Así lo explica la documentación:

> This gem includes a Rails concern called DeviseTokenAuth::Concerns::SetUserByToken. Include this concern to provide access to controller methods such as authenticate_user!, user_signed_in?, etc.

En `ApiController` se incluye el concern que ofrece la gema:

    class ApiController < ActionController::API
      include DeviseTokenAuth::Concerns::SetUserByToken
    
      before_action :authenticate_account!

De esa inclusión es donde proviene el método `current_account`.

Los docs de la gema explican que no es necesario que se incluya en el modelo:

    Typical use of this gem will not require the use of any of the following model methods. All authentication should be handled invisibly by the controller concerns.

Así que solo con lo del controlador es suficiente.

# Las Rutas y URLs

## Para ver el ExerciseProgram

La ruta en backend:

    GET /api/v1/external/exercise_programs/:id

La URL que será generada para el paciente:

    https://DOMAIN/evw/patients/EVW_TOKEN/exercises_programs/EXERCISE_PROGRAM_ID

Ejemplo en alpha:

    https://api2.alpha.getluna.com/evw/patients/814af859243ec8595c33/exercise_programs/4148c1ce-4047-4874-80e3-00d2bc943b91

La petición desde frontend

    GET /api/v1/external/exercise_programs/EXERCISE_PROGRAM_ID?token=WEB_TOKEN


## Para generar el enlace

La ruta en backend:

    POST /api/v3/therapist/exercise_programs/:id/send_link_to_patient

Como esto es para uso desde la app móvil, debería tenerse cargado tanto el `Patient` como el `ExerciseProgram` desde donde se ubique el botón.

Teniendo eso en cuenta, la petición debería ser así:

    POST /api/v3/therapist/exercise_programs/:id/send_link_to_patient?patient_id=PATIENT_ID

El proceso ubicará el `Account` de dicho `Patient` para poner su ID en la URL:

    https://DOMAIN/evw/patients/ACCOUNT_ID/exercises-programs/EXERCISE_PROGRAM_ID


# Entity-Relationship Diagram

Ver [[[EvW]_ER_Diagram]]

# Correr servidor Rails en local

Los servidores a correr son:

    be rails server # rails server
    
    ./bin/webpack-dev-server # webpack server para la UI
