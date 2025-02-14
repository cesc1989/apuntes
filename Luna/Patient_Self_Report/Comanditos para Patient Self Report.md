# Comanditos para Patient Self Report

Comandos rakes o para usar en la consola de Rails

## Buscar Intake o Progress incompletos

```ruby
PatientSelfReport::Form.onboarding.uncompleted.order("RANDOM()").first.link
PatientSelfReport::Form.progress.uncompleted.order("RANDOM()").first.link
```

## Revisar el estado de un form

Este sirve más para el API V2 de Mobile Progress Forms.

```ruby
bundle exec rake form:status:check[ace6143f-1082-4eee-b0f6-404bdc19b4ef]
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