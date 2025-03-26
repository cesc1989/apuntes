# RFC: Credentialing Form Extended Sign Up

When Therapists visit the `/sign-up` page and submit the form with their email address they get one of two outcomes:

1. They successfully sign the contract and can continue to the Credentialing Form
2. They are unable to sign the contract because the email is already in use

The second situation happens because, at the database level, we enforce a constraint to prevent a single email address to be used multiple times. This means that an email address that, for some reason, was previously used to sign a contract, cannot be used again in case a Therapist returns or if they were created in Hubspot with a different email address.

To circumvent this blocker, therapists use the same email address but add a suffix to make it "unique". For example, "takenemail+1@gmail.com". The problem with this work around is that it creates a new Contact in Hubspot which later have to be merged to the original intended Contact. This create a lot of manual work.

Now that Attestation Form and the new Hubspot Object Model are released we can provide a decent solution to this email problem. The solution relies on using the Hubspot Credentialing Object.

## Extended Sign Up Flow

This is how the whole process will pan out once the proposed implementation is released.

![[af.extended.signup.png]]

The Extended Sign Up will happen whenever a Therapist submits the contract with an email address existing in the Therapist DB. The overall process can be described like this:

- Look up HS Contact with provided email address
- Look up Therapist DB record with provided email address
- Compare the two results
- If they do not match:
	- Return an error to the therapist
	- Log the error
	- Report the error to Sentry
- If the results match:
	- Trigger a background job to start the Extended Sign Up
	- Return a 2xx response to indicate success

In the Extended Sign Up:
- Create a new Credentialing Object record for the HS Contact
- Save the new Credentialing Object record ID in the Therapist DB
- Associate HS Credentialing to HS Contact
- Attach the corresponding Association Label to the Credentialing Object record

# Details for Extended Sign Up

## Handling Existing Email Address

Instead of returning an error to the therapist indicating the email is already in use, the Extended Sign Up will take try to determine whether this process can be triggered or if there's an important discrepancy.

The idea is to *only* start the Extended Sign Up when the email address is taken *AND* there's a Hubspot Contact with that same email address.

If this condition is not met, we can provide an error message suggesting the therapist to reach out their Success Team rep. so they can escalate the issue to the dev team.

## Create New HS Credentialing Object Record



Q: Should the HS Contact be updated with the information from the Sign Up form?

## Save Newly created HS Credentialing Record ID


## Associate HS Credentialing Record to HS Contact


## Attach Corresponding Association Label to HS Credentialing Record


Q: What is the association label to attach?