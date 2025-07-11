# Custom Objects: Cuándo se Crean

## Credentialing

**Credentialing custom object is ==created when "Initial Form Date" is known==.**

This means they *signed the contract*.

 The "Initial Form Date" value corresponds to the `signup_form_date` property internal name.  This value is set in the `HubspotContactService` class. This value is set right after they complete the Sign Up form at `/sign-up/` URL.

## Therapist Address

**Therapist Address custom object is ==created when "Initial Form Date" is known.==**

This means they *signed the contract*.

The "Initial Form Date" value corresponds to the `signup_form_date` property internal name.  This value is set in the `HubspotContactService` class. This value is set right after they complete the Sign Up form at `/sign-up/` URL.

## License

Main License object would be created by a Hubspot workflow.

> Yes, since we are receiving the Treating license info via contract sign up form, HubSpot will auto create a new treating license after creating a new Credentialing object for each new contract.

Additional licenses will be created using HS API.
