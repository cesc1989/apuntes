# Filling therapist_direct_access_entries table

Let' pull data from a HubSpot Contact to fill the proposed table:
```ruby
t.references :therapist
t.references :state
t.boolean :permitted
t.bigint :hubspot_id
t.datetime :effective_from
t.integer :association_label
```

## Through endpoints

To pull Credentialing objects for a HS Contact hit endpoint:
```
GET https://api.hubapi.com/crm/v3/objects/contacts/:contactid?associations=objectTypeId
```

Example for HS Contact with ID `101298279979` and objectTypeId `2-33642689` for Credentialing objects.

```
GET https://api.hubapi.com/crm/v3/objects/contacts/101298279979?associations=2-33642689
```

Response:
```json
{
  "id": "101298279979",
  "properties": {
    "createdate": "2025-02-22T00:41:28.056Z",
    "email": "francisco.quintero+7612@ideaware.co",
    "firstname": "Prueba Redi",
    "hs_object_id": "101298279979",
    "lastmodifieddate": "2025-06-12T16:45:45.972Z",
    "lastname": "Rect"
  },
  "createdAt": "2025-02-22T00:41:28.056Z",
  "updatedAt": "2025-06-12T16:45:45.972Z",
  "archived": false,
  "associations": {
    "p7712148_credentialings": {
      "results": [
        {
          "id": "24387600522",
          "type": "contact_to_credentialings"
        },
        {
          "id": "24435403289",
          "type": "contact_to_credentialings"
        },
        {
          "id": "29446666061",
          "type": "contact_to_credentialings"
        },
        {
          "id": "29446666061",
          "type": "active_attested"
        },
        {
          "id": "24435403289",
          "type": "processing_for_move"
        },
        {
          "id": "24387600522",
          "type": "active"
        }
      ]
    }
  }
}
```

In the associations key we can find the Credentialing object ID for every association the Contact has.

By using one of those ids we can pull properties for the custom object. The endpoint to do that is:
```
GET https://api.hubapi.com/crm/v3/objects/:objectType/:objectId?properties=
```

Where objectTypeId corresponds to the Credentialing object schema ID (same as before) `2-33642689` and objectId is the Credentialing object from the previous results. Let's use the one with the "active_attested" type: `29446666061`.

```
GET https://api.hubapi.com/crm/v3/objects/2-33642689/29446666061?properties=therapist_name_state,direct_access_state,state,direct_access_restricted
```

The request includes in the query param `properties` the list to be returned in the response.

Response:
```json
{
  "id": "29446666061",
  "properties": {
    "direct_access_restricted": null,
    "direct_access_state": null,
    "hs_createdate": "2025-06-12T16:45:34.692Z",
    "hs_lastmodifieddate": "2025-06-12T16:46:10.264Z",
    "hs_object_id": "29446666061",
    "state": null,
    "therapist_name_state": "Prueba Redi Rect - CA"
  },
  "createdAt": "2025-06-12T16:45:34.692Z",
  "updatedAt": "2025-06-12T16:46:10.264Z",
  "archived": false
}
```

From this response we get possible values for:

- `state` field: `direct_access_state`, `state` or `therapist_name_state` (by extracting the state abbreviation)
- `permitted`: by parsing `direct_access_restricted`

## Through workflow/webhook

Create a workflow that detects when a Contact is associated to a Credentialing object of every label.

### Caveats

There is no way to extract the label from properties list or workflow options. ~~This means we'll need a workflow per association label. Each workflow would define it's own webhook to set the `association_label` column from each URL.~~

> [!Note]
> HubSpot workflows are very comprehensive and allow a lot of setups.
> A way to workaround not having the label as a property is by setting up branches per each association label. Each branch would send info to the same webhook and the label can be hardcoded per branch.

![[hs.workflow.branch.per.credentialing.png]]

---

I could not find a property for the Therapist ID in Luxe. To complete the record the `therapist_id` value would be inferred by looking the therapist using the `record_id` from HubSpot.

---

### Workflow Tests

The workflow in alpha is setup to trigger a webhook when a ==Contact is associated to any Credentialing object== and the ==Credentialing object has a value in the Associated Contact Record ID property==.

To be able to distinguish each Credentialing the workflow defines four branches for each possible label. Each branch will trigger the webhook sending properties:

- `state`
- `hs_object_id`
- `hs_createdate`
- `direct_access_state`
- `therapist_name_state`
- `direct_access_restricted`
- `associated_contact_record_id`

To be able to pull the label from each Credentialing it is send in the webhook URL as `label` query param.

In each branch's webhook setup, Credentialing records per label are made available. This way we guarantee the webhook with corresponding query param will correspond to the associated Credentialing kind.

#### Example for New Therapist Sign Up - Cred. Active Attested

After completing a sign up, got this from the webhook:

```ruby
"label: active-attested"
"state: CA"
"hs_object_id: 31006455396"
"hs_createdate: 1752778944619"
"direct_access_state: "
"therapist_name_state: Coshinita Kuintero - California"
"direct_access_restricted: "
"associated_contact_record_id: "
"label: active-attested"
```

#### Example manual association of Cred. Inactive

**First Test**

Tried manually adding a free Credentialing object to an existing Contact. Attached the Inactive label but the webhook never arrived.

**Second Test**

After enrolling all Contacts that meet criteria, tried again adding a manual Credentialing as Inactive but the webhook never arrived.


#### Enroll all Contacts after updating workflow

Update the workflow to enroll al Contacts and got multiple hits:

```json
"label: active-attested"
"state: CA"
"hs_object_id: 30674425623"
"hs_createdate: 1752266136983"
"direct_access_state: "
"therapist_name_state: Sarah Moving-Therapist-Test - Washington - California"
"direct_access_restricted: "
"associated_contact_record_id: "
"label: active-attested"

"label: active"
"state: "
"hs_object_id: 29590144058"
"hs_createdate: 1750096794740"
"direct_access_state: "
"therapist_name_state: JoseTEST RedirectTEST3 - Oregon"
"direct_access_restricted: "
"associated_contact_record_id: 129832437269"
"label: active"
```

Notice how the "active-attested" one does not include a value for `associated_contact_record_id`.