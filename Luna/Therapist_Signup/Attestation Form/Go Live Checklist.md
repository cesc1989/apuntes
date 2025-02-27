# Go Live Checklist for Attestation Form

Things that need to be done or need to be checked before going live.

## ✅ Run rake task that saves Credentialing-License association labels IDs to the settings table

```
bundle exec rake hubspot:associations:save_credentialing_to_license_labels
```

## ✅ Run rake task that saves Credentialing-Contact association labels IDs to the settings table

```
bundle exec rake hubspot:associations:save_credentialing_to_contact_labels
```

## ✅ Check all settings for Custom Objects exist

Check Custom Objects schemas exist in DB.
```
bundle exec rake hubspot:attestation:check_hs_custom_object_schemas
```

There should be 3 `setting` records for each custom object.

Check Custom Objects schemas in Hubspot.
```
bundle exec rake hubspot:attestation:check_custom_object_schemas_in_hubspot
```

The output of this one will help to compare with the output of the "Check Custom Objects schemas exist in DB."

## ✅ Check all settings for Custom Objects Associations labels exist

This rake should output all setting records storing IDs for association labels used in this relations:

- Credentialing-License
- Credentialing-Contact

Rake:
```
bundle exec rake hubspot:attestation:check_hs_custom_object_schemas
```

## Create flag in Omega

```ruby
Setting.find_or_create_by(key: "sync_to_hs_custom_objects", value: "true", setting_type: "other_text")
```


# Post Release

## Backfill AF URL in Hubspot Contacts

```
bundle exec rake hubspot:attestation:backfill_af_url
```

## Remove flag from Ruby classes that make requests to update HS Custom Objects

The flag is:
```ruby
Setting.find_by(key: "sync_to_hs_custom_objects")
```