# Grafana Queries

Queries para usar en los diferentes proyectos habilitados en Grafana.

## Clinical Dashboard

Para alpha u omega
```
{app="api-provider-portal"} != `Started GET "/dashboard"` != `Processing by DashboardController` != `Completed 200 OK` != `Sending envelope with items`
```

El operador `!=` hace que el texto que le sigue sea ignorado al buscar en los logs por lo cual el resultado final es más limpio (tendría menso ruido).

## Therapist Signup

Esta debería servir para alpha y omega.
```
{app="therapist-credentialing-backend"} != `path=/okcomputer/all.json` != `schema_migrations` != `SELECT "questions".* FROM "questions"` != `UPDATE "answers" SET "content"` != `SELECT "therapists".* FROM "therapists"` != `HTTP Origin header` != `INSERT INTO "answers"`
```

## Patient Self Report

Query que limpia bastante ruido de consultas de tablas que normalmente no sirven para debuggear algo:
```
{app="patient-self-report"} != `SELECT "questions".* FROM "questions"` != `HTTP Origin header` != `Can't verify CSRF token authenticity.` != `Answer Pluck` != `Answer Count` != `Form Load` != `Answer Load` != `Question Pluck` != `PainSpot Pluck`
```