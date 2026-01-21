# General Flow to Create Forms

Etiquetas: #patient_forms_creation

In this document I'll list all places I can find where intake forms are created and read from.

## Intake Form Creation

> [!Note]
> After AppBlend, now Patient Self Report lives in backend so it's no longer an external service. Everything happens between `backend` and `Marketplace`.

Form creation starts from Backend, goes through Marketplace, for it to finally happen in Patient Self Report, to return the final results to Marketplace where it's saved to the database.

> [!Important]
> See these two from Marketplace:
> 
> - [[Creación_de_intake_form en Marketplace]]
> - [[Intake_Form_and_Progress_Form_creation_flow en Marketplace]]

This flow provides an overview of this process:

![[50.intakeform.creation.flow.png]]

# Accessing Existing Forms

See [[Access Existing Forms]]