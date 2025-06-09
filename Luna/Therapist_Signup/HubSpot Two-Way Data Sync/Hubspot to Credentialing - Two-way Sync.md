# Hubspot to Credentialing Form: Two-way Sync

Let's add a webhook to allow Hubspot to update Therapist information in the Therapist Signup DB.

## How are Hubpspot Webhooks done in Edge

There's multiple webhooks setup in Edge for two-way syncing with Hubspot.

See:

- `DraftTherapistController`
- `Api::V1::External::PatientWaitlistRemovalHubspotWebhooksController`

Also see `IncomingHubspotWebhookController` which holds code for filters, auth with Hubspot, and HS properties checks.

This controller is used as parent and has methods like:

- `valid_hubspot_signature?`: which checks the incoming request is indeed from Hubspot.
- `valid_mandatory_property?`: to check a property is blank or not.

# Webhook definition

Considering therapist information is spread out in multiple Custom Objects (Contact, Credentialing, Therapist Address, License) the webhook will need to account for that in the payload it'll receive and also to what DB record to update.

Besides auth headers, the webhook should receive these properties in the payload:

- `object_id`: this is the ID of the Custom Object
- `record_id`: this is the ID of the custom object record
- `properties`: hash of the updated properties

Example. For a Credentialing object where the object itself has ID `2-33642689` and any record would have the ID `25685568368`, a request payload could be:
```json
{
  "object_id": "2-33642689",
  "record_id": "25685568368",
  "properties": {
    "attestation_form_submitted_date": "12/31/2024"
  }
}
```

## Case for `object_id` property

The goal here is to be able to compare the receive ID with any of the setting records storing `_objectTypeId`.

Once the object owning the properties is identified, we can distribute incoming properties to corresponding records in the DB.

This needs a table to match Custom Objects properties to DB tables and columns.

# Example Webhook Payload

This is seen in some Edge tests:
```ruby
{
  "vid" => "1234",
  "properties" => {
    "therapist_or_patient_" => { "value" => "Therapist" },
    "firstname" => { "value" => "Squirty" },
    "lastname" => { "value" => "McFlurry" },
    "email" => { "value" => "Squirty@McFlurry.com" },
    "gender" => { "value" => "male" },
    "phone" => { "value" => "+1 (555) 555-5555" },
    "date_of_birth" => { "value" => "651033111000" },
    "desired_weekly_appointments" => { "value" => "5" },
    "desired_signature" => { "value" => "JT, DD" },
    "pet_allergies" => { "value" => "yes" },
    "bank_account_type" => { "value" => "business" },
    "specialties_credentialing_form" => { "value" => "Geriatrics,Neurological,CRPS" },
    "address" => { "value" => "123 Fake St., #333" },
    "city" => { "value" => "Oakland" },
    "state" => { "value" => "California" },
    "zip" => { "value" => "94102" },
    "treating_street_address" => { "value" => "650 Gough Street West, Apt A" },
    "treating_city" => { "value" => "San Francisco" },
    "treating_state" => { "value" => "California" },
    "treating_postal_code" => { "value" => "94555" }
  }
}
```