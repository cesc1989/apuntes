# 2025-06-09 - P1, Clinical Dashboard Patient Data Download Inaccurate

## Resumen

El 9 de Junio llegó un reporte de Help Desk (por Nicolas Francoeur) que los datos descargados desde la sección "Patients Treated" para el provider MHS contenía datos de todos los pacientes no solo los de MHS.

## Nivel de Severidad

P1

## Tiempos

TTD: 1:45min
TTE: 2:24min
TTF: 25hrs (next day)

## 3 - 5 Aprendizajes

- Se necesitan más pruebas exhaustivas alrededor de características críticas como esta.

## Evaluación de Impacto

No hubo impacto para los pacientes de Luna.

Dos ingenieros intervinieron para resolver este problema. Francisco Q. y Fabricio.

## Monitoreo

El problema fue descubierto por el reporte en Help Desk.

## Análisis de la causa raíz

### Cinco Por Qué

1. What caused this outage to happen?
A feature was providing all Clinical Dashboard patients data to any Clinic provider requesting it.

2. What caused (1) to happen?
The feature was not set to accept patient data download for Clinics.

3. What caused (2) to happen?
Oversight in manual testing this specific feature when Clinic providers started being supported in Clinical Dashboard.

4. What caused (3) to happen?
Alpha limits make us something skip some testing because there's not enough good data to test.

5. What caused (4) to happen?
Clinical Dashboard data is sourced from specific queries that limit the amount of data available to play with in Alpha.

### Cómo pudo haberse...

Prevenido:

Detectado más temprano:

Mitigado más pronto:

## Línea Temporal

All times are in UTC-5

June 9th

10:44am - Help Desk report
12:28pm - Ryan intervenes
12:46pm - Alexis intervenes
1:09pm - Francisco engages after standup notice
4:34pm - Fabricio puts frontend update
7:36pm - Francisco put backend update in Omega. Pending tests in alpha.

June 10th

10:00am - Francisco tested in alpha. Issue persisted.
10:59am - Francisco submitted and teste fixed. Asked Nicolas to try again in Omega.
11:13am - Nicolas confirmed issue was fixed.