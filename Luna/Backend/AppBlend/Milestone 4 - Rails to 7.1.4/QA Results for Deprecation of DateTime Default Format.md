# QA Results for Deprecation of DateTime Default Format

After disabling the deprecation for the support of the default format for Date and DateTime interpolation, date fields/columns would display the date value in a scrambled format.

When displaying edit form fields, the date is shown scrambled. For example, this _"20/25/0131"_ instead of _"01/31/2025"_.

When displaying in regular pages, the date is not shown in the American date format. For example, this _"2025-01-31"_ instead of _"01/31/2025"_.

The following are all of the places where this problem happens and how they look after applying the adequate corrections.

## Benefits Verification - Edit Form

Affected fields: `date_of_birth`.

![[30.benefits.verifications.png]]

## Payer Authorizations - Edit Form

Affected fields: `effective_until`, `submitted_at`.

![[31.payer.auths.png]]

## Physicians - Edit Form

Affected fields: `verification_sent_at`.

In both Escalation and Portal recipients sections.

![[32.physicians.edit.png]]

## Care Plans - Edit Form

Affected fields: `effective_until`, `surgery_date`.

![[33.careplans.edit.png]]

## Pathway Assignments - List Page

Affected fields: `surgery_date`.

![[34.pathway.assigns.png]]

## Protocol Escalations - List Page

Affected fields: `visit_date`.

![[35.protocol.escalations.png]]

## Care Plan Route Reviews - List Page

Affected fields: IV column.

![[36.careplan.route.reviews.png]]

## Credentialing Entries - List Page

Affected fields: `effective_from`.

![[37.cred.entries.png]]

## Clinic Payers - Detail Page

Affected fields: `effective_from`.

![[38.clinic.payers.detail.png]]
