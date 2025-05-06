# Backfill Attestation Form URL to Contacts

As part of the release of the Attestation Form, a backfill rake task to fill the Hubspot Contact property `attestation_form_url` was added and ran. It was expected to not complete successfully. Now I have to complete this work.

What follows is context to understand the current state, where to pick it up, and what to do next.

## Context

This rake was ran on Feb 27th:
```bash
bundle exec rake hubspot:attestation:backfill_af_url
```

It failed at some point. Bad thing is that by the output shared it's not clear at what point:
```bash
I, [2025-02-27T22:51:44.417518 36] INFO -- : Backfill AF URL for therapist with hubspot_id: rake aborted! I, [2025-02-27T22:51:44.431025 36] INFO -- sentry: [Transport] Sending envelope with items [event] 897973b7c4c447059735b80afc8875a2 to Sentry Hubspot::InvalidParams: expecting Integer or Array of Integers parameter /usr/local/bundle/gems/hubspot-ruby-0.9.0/lib/hubspot/contact.rb:100:in `find_by_id' /usr/src/app/lib/tasks/022_backfill_attestation_form_url.rake:12:in `block (5 levels) in <main>' /usr/local/bundle/gems/retriable-3.1.2/lib/retriable.rb:61:in `block in retriable' /usr/local/bundle/gems/retriable-3.1.2/lib/retriable.rb:56:in `times' /usr/local/bundle/gems/retriable-3.1.2/lib/retriable.rb:56:in `retriable' /usr/local/bundle/gems/retriable-3.1.2/lib/retriable.rb:22:in `with_context' /usr/src/app/lib/tasks/022_backfill_attestation_form_url.rake:10:in `block (4 levels) in <main>' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:71:in `each' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:71:in `block in find_each' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:138:in `block in find_in_batches' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:245:in `block in in_batches' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:229:in `loop' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:229:in `in_batches' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:137:in `find_in_batches' /usr/local/bundle/gems/activerecord-7.0.4/lib/active_record/relation/batches.rb:70:in `find_each' /usr/src/app/lib/tasks/022_backfill_attestation_form_url.rake:7:in `block (3 levels) in <main>' /usr/local/bundle/gems/sentry-ruby-5.11.0/lib/sentry/rake.rb:26:in `execute' /usr/local/bundle/gems/rake-13.0.6/exe/rake:27:in `<top (required)>' /usr/local/bundle/bin/bundle:23:in `load' /usr/local/bundle/bin/bundle:23:in `<main>' Tasks: TOP => hubspot:attestation:backfill_af_url (See full trace by running task with --trace)
```

The last few successful ones:  
```
I, [2025-02-27T22:51:42.863256 #36]  INFO -- : Did not find Hubspot Contact for ID: 234277151
I, [2025-02-27T22:51:42.863396 #36]  INFO -- : Backfill AF URL for therapist with hubspot_id: 152122701
I, [2025-02-27T22:51:43.353865 #36]  INFO -- : Backfill AF URL for therapist with hubspot_id: 7641059603
I, [2025-02-27T22:51:43.898151 #36]  INFO -- : Backfill AF URL for therapist with hubspot_id: 100346821525
```

## The property `attestation_form_url`

In Omega, the property is only used in 8 contacts.

> 0.49% 6,721 out of 1,385,470

In Alpha, it's just 1 contact.

> 0.06% 103 out of 177,386

This is all very off considering the rake was ran both in my local end and in alpha.

> [!Important]
> This property also exists for the HS Credentialing object.

## What to do next?

✅ Add a new field to the `therapists` table: `backfilled_attestation_form_url`. Boolean default to false.

- After a HS Contact is updated, set this field to true.
- Use this as a flag to be able to reduce the amount of therapists to query when rerunning the task.

✅ Capture and log more errors in different steps of the process.