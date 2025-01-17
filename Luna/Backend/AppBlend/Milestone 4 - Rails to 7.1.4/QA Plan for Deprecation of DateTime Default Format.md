# QA Plan for Deprecation of DateTime Default Format

After disabling the deprecation for the support of the default format for Date and DateTime interpolation, date fields/columns would display the date value in a scrambled format.

When displaying edit form fields, the date is shown scrambled. For example, this _"20/25/0131"_ instead of _"01/31/2025"_.

When displaying in regular pages, the date is not shown in the American date format. For example, this _"2025-01-31"_ instead of _"01/31/2025"_.

The following are all of the places where this problem happens and how they look after applying the adequate corrections.

## Benefits Verification - Edit Form

Affected fields: `date_of_birth`.

## Payer Authorizations - Edit Form

Affected fields: `effective_until`, `submitted_at`.

## Physicians - Edit Form

Affected fields: `verification_sent_at`.

In both Escalation and Portal recipients sections.

## Care Plans - Edit Form

Affected fields: `effective_until`, `surgery_date`.

## Pathway Assignments - List Page

Affected fields: `surgery_date`.

## Protocol Escalations - List Page

Affected fields: `visit_date`.

## Care Plan Route Reviews - List Page

Affected fields: IV column.

## Luna Fax Numbers - List Page

Affected fields: `created_at`, `updated_at` columns.

## Credentialing Entries - List Page

Affected fields: `effective_from`.

## Clinic Payers - Detail Page

Affected fields: `effective_from`.