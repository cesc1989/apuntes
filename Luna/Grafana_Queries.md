# Grafana Queries

Etiqueta #grafana

Queries para usar en los diferentes proyectos habilitados en Grafana. Más que nada están definidas para limpiar contenido de los logs y poder enfocar lo que necesito.

## Sintaxis

El operador `!=` hace que el texto que le sigue sea ignorado al buscar en los logs por lo cual el resultado final es más limpio (tendría menso ruido).

## Clinical Dashboard

Para alpha u omega
```sql
{app="api-provider-portal"} != `Started GET "/dashboard"` != `Processing by DashboardController` != `Completed 200 OK` != `Sending envelope with items`
```

## Therapist Signup

Esta debería servir para alpha y omega.
```sql
{app="therapist-credentialing-backend"} != `path=/okcomputer/all.json` != `schema_migrations` != `SELECT "questions".* FROM "questions"` != `UPDATE "answers" SET "content"` != `SELECT "therapists".* FROM "therapists"` != `HTTP Origin header` != `INSERT INTO "answers"`
```

Esta sirve cuando quiero revisar el registro de algún therapist. Limpio todo el ruido posible y solo dejo lo del endpoint signup más algunas cosas.
```sql
{app="therapist-credentialing-backend"} != `path=/okcomputer/all.json` != `schema_migrations` != `SELECT "questions".* FROM "questions"` != `UPDATE "answers" SET "content"` != `SELECT "therapists".* FROM "therapists"` != `HTTP Origin header` != `INSERT INTO "answers"` != `INSERT INTO "credentialing_informations"` != `INSERT INTO "work_records"` != `INSERT INTO "immunizations"` != `INSERT INTO "preferences"` != `INSERT INTO "npi_and_caqh_applications"` != `INSERT INTO "professional_histories"` != `INSERT INTO "personal_references"` != `INSERT INTO "medicare_requirements"` != `INSERT INTO "payouts"` != `Can't verify CSRF token authenticity` != `SELECT` != `UPDATE` != `[ActiveJob]` != `Therapist Update` != `[REFERENCES]` != `TRANSACTION` != `method=GET` != `method=PUT` != `[HUBSPOT]` != `File Upload` != `[DATE_TRANSFORMATION]` != `Warning File upload` != `class=Hubspot` != `class=ProfessionalReference` != `Couldn't connect reference for` != `class=ActionMailer`
```

Esta es solo para limpiar ruido. No deja nada.
```sql
{app="therapist-credentialing-backend"} != `ActiveRecord::SchemaMigration Pluck` != `okcomputer/all.json` != `Therapist Load` != `Alias Load` != `WorkRecord Load` != `WorkGap Load` != `Immunization Load` != `CredentialingInformation Load` != `Payout Load` != `HTTP Origin header` != `DATE_TRANSFORMATION` != `Question Load` != `Setting Load` != `Warning File` != `License Load` != `Preference Load` != `Answer Load` != `MedicareRequirement Load` != `[HUBSPOT] sync` != `path=/v1/cities` != `path=/v1/states` != `path=/v1/countries` != `path=/v1/therapists` != `path=/v1/questions` != `HubspotPersonalInformationWorker` != `HubspotImmunizationWorker` != `HubspotCredentialingInformationWorker` != `HubspotProfessionalHistoryWorker` != `HubspotPreferenceWorker` != `SELECT "professional_histories"` != `Answer Pluck` != `PersonalReference Load` != `License Exists?` != `License Pluck` != `NpiAndCaqhApplication Load` != `Answer Update` != `HubspotMedicareRequirementWorker` != `File Upload` != `Therapist Update` != `HubspotProfessionalReferenceWorker` != `Preference Update` != `HubspotNpiAndCaqhWorker` != `INSERT INTO "answers"` != `COMMIT` != `BEGIN` != `HubspotPayoutWorker` != `sidekiq-7.0.2` != `Alias Count` != `ProfessionalHistory Update` != `ServiceState Load` != `activesupport` != `CredentialingInformation Create` != `Therapist Create` != `HubspotSubmitFormWorker` != `WARN --`
```


## Patient Self Report

> [!Note]
> Patient Self Report existe dentro de backend así que la app es edge.

```sql
{app="edge"} != `SELECT "questions".* FROM "questions"` != `HTTP Origin header` != `Can't verify CSRF token authenticity.` != `Answer Pluck` != `Answer Count` != `Form Load` != `Answer Load` != `Question Pluck` != `PainSpot Pluck`
```