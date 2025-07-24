# Permitted or Restricted

Notas y pistas para tener claro el significado de la bandera.

En la tabla el campo quedó definido como:
```ruby
t.boolean :permitted, default: false, null: false
```

El ERD aclara su uso esperado:
```ruby
permitted -- boolean [true = DA Permitted; false = DA Restricted]
```

Esto dice el ERD sobre cómo manejar la bandera:
> Despite “Direct Access Restricted” and “Direct Access Permitted” (...) are all the same idea (...) `false` it means the therapist is restricted to only patients with a referral (...) `true` (...) therapist is not restricted to patients with a referral.

Le pedí una tabla de lógica a Claude según el código que hizo. Ver en [[permitted_field_logic]]

> [!Note]
> Si `direct_access_restricted` dice "Yes..." es que está restringido.


# Filling `therapist_direct_access_entries` table

Let' pull data from a HubSpot Contact to fill the proposed table:
```ruby
t.references :therapist
t.references :state
t.boolean :permitted
t.bigint :hubspot_id
t.datetime :hubspot_created_at
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

- `state`: property `state` or `therapist_name_state` (by extracting the state abbreviation)
- `permitted`: by parsing property `direct_access_restricted`

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
- `associated_contact_record_id`
- `hs_createdate`
- `direct_access_state`
- `direct_access_restricted`
- `therapist_name_state`

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

# Two Properties: Direct Access State & Direct Access Restricted

Found those two properties in the Credentialing object properties list:

**Direct Access Restricted `direct_access_restricted`**

Description: _Captures if the therapist is direct access restricted or able to treat patients unrestricted._

Has multiple values:
- No
- "Yes: MN < 1 year experience"
- "Yes: NY < 3 years experience"

**Direct Access State `direct_access_state`**

Description: _Identifies if the therapist can practice under direct access._

Has values: Yes/No.

## Which one to use?

Asked Jessica and told to use Direct Access Restricted:
> Direct Access Restricted - and use the one in the credentialing object.  
The state was more relevant when it was on the contact since the state could change

# Why Claude wrote `path.dup` for Requests

Este código:
```ruby
props = "therapist_name_state,direct_access_state,state,direct_access_restricted,associated_contact_record_id"
path = "/crm/v3/objects/:object_type/:object_id"
params = {
  object_type: object_type_id,
  object_id: object_id,
  properties: props
}

Hubspot::Connection.get_json(path.dup, params)
```

Le pregunté por qué era necesario el `path.dup`.

## The Problem

The Hubspot::Connection.get_json method internally calls generate_url(path, opts) which:

1. Tries to clone the path: `path = path.clone`
2. Mutates the string in-place: Uses `path.gsub!(":#{k}", CGI.escape(v.to_s))` to replace interpolation placeholders like `:object_type`, `:id`, etc.

However, clone only creates a shallow copy, so the string content still gets mutated.

### Example Without .dup

`path = "/crm/v3/objects/:object_type/:object_id"`

```ruby
# First call
Hubspot::Connection.get_json(path, object_type: "contacts", object_id: 123)
# path is now "/crm/v3/objects/contacts/123" - placeholders replaced!

# Second call fails
Hubspot::Connection.get_json(path, object_type: "deals", object_id: 456)
# Error: no :object_type or :object_id placeholders left to replace
```

### The Solution

Using `path.dup` creates a new string object each time, protecting the original:

```ruby
path = "/crm/v3/objects/:object_type/:object_id"

Hubspot::Connection.get_json(path.dup, object_type: "contacts", object_id: 123)
Hubspot::Connection.get_json(path.dup, object_type: "deals", object_id: 456)

# Both calls work because original path remains unchanged
```

This is a defensive programming practice to work around the gem's URL interpolation bug where the path parameter gets mutated during processing.

### Is this because of magick string?

The `.dup` is needed regardless of the `# frozen_string_literal: true` comment. Here's why:

Two Different Issues

1. Frozen string literals: When enabled, string literals become frozen (immutable)
2. HubSpot gem bug: The gem mutates the path parameter during URL generation

What happens with/without frozen strings:

With frozen strings (`# frozen_string_literal: true`):
```ruby
path = "/crm/v3/objects/:object_type/:object_id"  # This string is frozen
Hubspot::Connection.get_json(path, params)  # Would raise FrozenError when gem tries to mutate
```

Without frozen strings:
```ruby
path = "/crm/v3/objects/:object_type/:object_id"  # String is mutable
Hubspot::Connection.get_json(path, params)  # Mutates path, breaking reuse
```

Why .dup fixes both cases:
```ruby
path = "/crm/v3/objects/:object_type/:object_id"
Hubspot::Connection.get_json(path.dup, params)  # Creates new mutable string
```

The .dup:
- Creates a mutable copy (fixes frozen string issue)
- Protects the original from mutation (fixes reuse issue)

So the `.dup` would be necessary even without the frozen string literal comment - it's solving the underlying gem bug, not just working around frozen strings.