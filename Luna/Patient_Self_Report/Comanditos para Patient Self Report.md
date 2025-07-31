# Comanditos para Patient Self Report

Comandos rakes o para usar en la consola de Rails.

## Buscar Intake o Progress incompletos

Intake Form:
```ruby
PatientSelfReport::Form.onboarding.uncompleted.order("RANDOM()").first.link
```

Progress Form:
```ruby
PatientSelfReport::Form.progress.uncompleted.order("RANDOM()").first.link
```

## Revisar el estado de un form

Este sirve más para el API V2 de Mobile Progress Forms.

```ruby
bundle exec rake form:status:check[ace6143f-1082-4eee-b0f6-404bdc19b4ef]
```

## Resetear un Form y poner todo en orden

Solo resetear no crearía los Answers ni AggravatingActivities.
```ruby
form = PatientSelfReport::Form.find_by(uuid: [UUID_IN_URL])
form.reset!

PatientSelfReport::InitialAnswers.new(form).create
PatientSelfReport::InitialAggravatingActivities.new(form).create
```

## Crear Initial Answers o Aggravating Activities de  un form

Para forms viejos o si falló el worker.

```ruby
PatientSelfReport::InitialAggravatingActivities.new(Form.find(1603)).create
PatientSelfReport::InitialAnswers.new(Form.find(32314)).create
```

## Buscar UserCommunicationMethod (UCM) para probar Email Verification

Estos sirven para probar Portal Recipients y que cargue la página de verificar o de verificado.

```ruby
ucm = UserCommunicationMethod.email.find("291f69bb-a3d5-4dbd-ab41-64bec1cbe56d")
ucm.update(
  verification_code:nil,
  verification_code_expires_at:nil,
  verification_sent_at:nil,
  verification_attempts: 0,
  verification_status: :unverified
)
ucm.update(verification_status: :verified)
```

## Programa worker que envía email de Verificación a un UCM

```ruby
UserCommunicationMethods::EmailVerificationReminderWorker.perform_in(500, "291f69bb-a3d5-4dbd-ab41-64bec1cbe56d")
```

## Variados para encontrar Pacientes del subdominio Patient Self Report

```ruby
PatientSelfReport::Patient.find_by(internal_id: "714920d5-4707-4b3a-914a-bf82c125300d")

PatientSelfReport::Patient.find(9207320329312777335)

PatientSelfReport::Patient.select(:id).max

PatientSelfReport::Patient.joins(:forms).where(email: nil).where(forms: { progress_type: "ongoing", completed: false })

PatientSelfReport::Form.find(24898).update_attribute(:completed, false)

PatientSelfReport::Form.find_by(uuid: "ab6b3654-1025-4ca7-99c5-cb60df682525").destroy
```

## Query para encontrar patients con más de dos forms

```sql
select patient_id, count(1) as cantidad
from forms
group by patient_id
having count(1) > 2;
```


# Otros Comanditos

El de Therapist Signup -> [[Comanditos para Therapist Signup]]