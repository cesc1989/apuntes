# Consent Fee Improvements

Autor: Alexis Ramis

Sistema: backend

## Contexto

El sistema para recoger el consentimiento del paciente era sencillo: un paciente aceptaba o no todos los términos de Luna.

Sin embargo, el sistema ha evolucionado:

- los términos son dinámicos
- un paciente puede tener varios care plans
- se han consolidado los servicios (appblend)

Esto trae una serie de problemas:

- se pierden datos
- se crean documentos duplicados
- se lleva registro incorrecto del consentimiento del paciente

## Problema

Hay tres problemas clave:

- Solo hay una forma de llevar el registro del consentimiento
	- solo se puede consentir una vez y aplica para todos los care plans
- Documentos duplicados
	- por la lambda
- Pacientes pueden completar el Intake Form form incluso con el Care Plan incompleto

## Objetivo

- Consentimiento que sea por Care Plan
- Eliminar la duplicación de documentos
- Consentimiento se recoge antes de completar el Intake Form

## Propuesta

### Migración

- Mover los campos de consent de la tabla de `patients` a `forms`.
	- Para que sea una relación de 1:1 entre dato de consent y care plan

### Simplificación de Documentos

- Quitar la lambda que genera el PDF de los términos del paciente
	- Ahora que Forms vive en backend no se necesita de petición entre servicios + lambda

### Validación de Intake Form

- Guardias para prevenir que se generen o envíen Intake Forms si el care plan está incompleto.

## Milestones

- M1: cambios del schema
	- tabla `forms` guarda el consentimiento
- M2: actualización del código
	- `backend` usa nuevo sistema para registrar el consentimiento
- M3: backfill
- M4: limpieza y borrado
	- Borrar la lambda y controladores