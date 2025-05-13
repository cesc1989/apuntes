# RFC: Credentialing Form Extended Sign Up

When Therapists visit the `/sign-up` page and submit the form with their email address they get one of two outcomes:

1. They successfully sign the contract and can continue to the Credentialing Form
2. They are unable to sign the contract because the email is already in use

The second situation happens because, at the database level, we enforce a constraint to prevent a single email address to be used multiple times. This means that an email address that, for some reason, was previously used to sign a contract cannot be used again in case a Therapist returns or if they were created in Hubspot with a different email address.

To circumvent this blocker, therapists use the same email address but add a suffix to make it "unique". For example, "takenemail+1@gmail.com". The problem with this work around is that it creates a new Contact in Hubspot which later have to be merged to the original intended Contact. This create a lot of manual work.

Now that the Attestation Form and the new Hubspot Object Model are released we can provide a decent solution to this email problem. The solution relies on using the Hubspot Credentialing Object.

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

==The idea is to *only* start the Extended Sign Up when the email address is taken *AND* there's a Hubspot Contact with that same email address.==

If this condition is not met, we can provide an error message suggesting the therapist to reach out their Success Team rep. so they can escalate the issue to the dev team.

## Create New HS Credentialing Object Record

To create a new HS Credentialing Object record, make a POST request to the V3 objects endpoint using the Credentialing Object ID. This ID exists in the Therapist DB in the settings table.

```ruby
Setting.find_by(key: "credentialings_id").value
```


```
POST https://api.hubspot.com/crm/v3/objects/[CREDENTIALING_OBJECT_ID]

Payload

{
  "properties": {
    "attestation_form_url": "https://theattestationformurl.com/id"
  }
}
```

### Q&A

Q: Should the HS Contact be updated with the information from the Sign Up form?

Q: What other properties should be set when creating the new Credentialing record?

## Save Newly created HS Credentialing Record ID

> [!Note]
> Creating the Credentialing record and saving the ID happen in the same moment but they're better described as separate steps to understand the overall process.

The request made in the previous step will return an ID property that should be saved to the Therapist DB. More precisely, in the `credentialing_hubspot_id` of the `therapists` table.

## Associate HS Credentialing Record to HS Contact

To connect the HS Contact with the new HS Credentialing record they need to be associated first. This can be done using the V4 Associations API.

To complete this step we'll make a POST request to the corresponding endpoint specifying the corresponding `associationTypeId` that indicates the direction of the association and the record IDs to be associated: Contact record ID and Credentialing Record ID.

Example:
```
POST https://api.hubapi.com/crm/v4/associations/credentialings/contact/batch/create

Payload

{
  "inputs": [
    {
      "types": [
        {
          "associationCategory": "USER_DEFINED",
          "associationTypeId": 62 // credentialings_to_contact_active_type_id
        }
      ],
      "from": {
        "id": "24454281832"
      },
      "to": {
        "id": "101935997007"
      }
    }
  ]
}
```

### Q&A

Q: What is the association label to attach to the new Credentialing record?

# Additional Considerations

## Regular Sign Up alongside Extended Sign Up

Extended Sign Up will happen in a background job branched off the regular Sign Up process. This means a new signed contract PDF file will be generated, emailed to the therapist, and uploaded to their S3 folder.

Any other thing that is done as part of a Regular Sign Up will be done as well after Extended Sign Up is triggered.

## Pre-populate email input field from URL

To minimize chances of therapists inputing an incorrect email address, let's pre-populate the email field with a value present as a query parameter in the Sign Up form URL.

For example, this URL:
```
https://success.alpha.getluna.com/sign-up/california?email=niceemail@gmail.com
```

will pre-fill the email input field with "niceemail@gmail.com".


# Tasks

- Change email validation step to not return error but look up records to compare
- Handle error when looked up records do not match
- Setup worker to kick start Extended Sign Up
- Add class to handle creating a new Credentialing object
- Add class to handle associating new Credentialing object to Contact
- Run manual tests in Alpha env