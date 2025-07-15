# Custom Objects: Cuándo se Crean

## Credentialing "Active Attested"

**Credentialing custom object is ==created when "Initial Form Date" is known==.**

This means they *signed the contract*.

 The "Initial Form Date" value corresponds to the `signup_form_date` property internal name.  This value is set in the `HubspotContactService` class. This value is set right after they complete the Sign Up form at `/sign-up/` URL.

## Credentialing "Processing for Move"

To initialize the Moving Therapist scenario, you have to follow these steps (for alpha):  

1. Fill out this form using the same email: [https://share.hsforms.com/1vTz2g38VSmGb7rWehrAG1A4laqc](https://share.hsforms.com/1vTz2g38VSmGb7rWehrAG1A4laqc)
2. Choose "I am moving to a new state" under moving information.
3. Submit it and wait 1-2 minutes for HubSpot to process address objects and create new Credentialing object with "Processing for Move" label


## Therapist Address

**Therapist Address custom object is ==created when "Initial Form Date" is known.==**

This means they *signed the contract*.

The "Initial Form Date" value corresponds to the `signup_form_date` property internal name.  This value is set in the `HubspotContactService` class. This value is set right after they complete the Sign Up form at `/sign-up/` URL.

## License

Main License object would be created by a Hubspot workflow.

> Yes, since we are receiving the Treating license info via contract sign up form, HubSpot will auto create a new treating license after creating a new Credentialing object for each new contract.

Additional licenses will be created using HS API.
