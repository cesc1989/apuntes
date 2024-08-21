# Grafana Queries

Queries para usar en los diferentes proyectos habilitados en Grafana.

## Clinical Dashboard

Para alpha u omega
```sql
{app="api-provider-portal"} != `Started GET "/dashboard"` != `Processing by DashboardController` != `Completed 200 OK` != `Sending envelope with items`
```

El operador `!=` hace que el texto que le sigue sea ignorado al buscar en los logs por lo cual el resultado final es más limpio (tendría menso ruido).

## Therapist Signup

Esta debería servir para alpha y omega.
```sql
{app="therapist-credentialing-backend"} != `path=/okcomputer/all.json` != `schema_migrations` != `SELECT "questions".* FROM "questions"` != `UPDATE "answers" SET "content"` != `SELECT "therapists".* FROM "therapists"` != `HTTP Origin header` != `INSERT INTO "answers"`
```

Esta sirve cuando quiero revisar el registro de algún therapist. Limpio todo el ruido posible y solo dejo lo del endpoint signup más algunas cosas.
```sql
{app="therapist-credentialing-backend"} != `path=/okcomputer/all.json` != `schema_migrations` != `SELECT "questions".* FROM "questions"` != `UPDATE "answers" SET "content"` != `SELECT "therapists".* FROM "therapists"` != `HTTP Origin header` != `INSERT INTO "answers"` != `INSERT INTO "credentialing_informations"` != `INSERT INTO "work_records"` != `INSERT INTO "immunizations"` != `INSERT INTO "preferences"` != `INSERT INTO "npi_and_caqh_applications"` != `INSERT INTO "professional_histories"` != `INSERT INTO "personal_references"` != `INSERT INTO "medicare_requirements"` != `INSERT INTO "payouts"` != `Can't verify CSRF token authenticity` != `SELECT` != `UPDATE` != `[ActiveJob]` != `Therapist Update` != `[REFERENCES]` != `TRANSACTION` != `method=GET` != `method=PUT` != `[HUBSPOT]` != `File Upload` != `[DATE_TRANSFORMATION]` != `Warning File upload` != `class=Hubspot` != `class=ProfessionalReference` != `Couldn't connect reference for` != `class=ActionMailer`
```


## Patient Self Report

Query que limpia bastante ruido de consultas de tablas que normalmente no sirven para debuggear algo:
```sql
{app="patient-self-report"} != `SELECT "questions".* FROM "questions"` != `HTTP Origin header` != `Can't verify CSRF token authenticity.` != `Answer Pluck` != `Answer Count` != `Form Load` != `Answer Load` != `Question Pluck` != `PainSpot Pluck`
```