# 🐞 Error tracking for Engagement Metrics

In Lucky Orange we detect many errors related to “Access Denied” S3 thing but they do not appear in the database, in the `Activity` table.

In this document I log what I find to try to understand and make the tracking more accurate.

## Notes

How to generate an expired presigned URL
```bash
aws s3 presign s3://luna-staging-patient-csv-downloads/metrics_export_1665063833.csv --expires-in 300
```


# Updates

One cause that errors weren’t being tracked was that validation to create the event failed silently. Updated in merged code.

# Logs

## In Lucky Orange

I reviewed recorded and tagged videos and found many errors.

- “Access denied” on POC: 4 times
- “Failed to fetch” when opening link: 6 times
- “Charts are way too skewed”: 1 times
    - The bars in the graphics do not expand
    - Could be a css issue or a lucky orange issue with recordings
- “Lucky orange failure”: 4 times
- Dashboard never loaded: 2 times
    - spinning circle stuck forever
- “error 500” from server: 2 times

## In Database

When asked to run a query, got this:
```ruby
date_range = Date.parse('01/01/2023')..Date.parse('31/01/2023')
jan_errors = Activity.where(kind: 'error').where(created_at: date_range)

pp jan_errors.count
pp jan_errors

irb(main):003:0> pp jan_errors.count
D, [2023-02-15T17:01:31.804143 #17159] DEBUG -- :    (2.9ms)  SELECT COUNT(*) FROM "activities" WHERE "activities"."kind" = $1 AND "activities"."created_at" BETWEEN $2 AND $3  [["kind", "error"], ["created_at", "2023-01-01"], ["created_at", "2023-01-31"]]

2

irb(main):004:0> pp jan_errors
D, [2023-02-15T17:01:34.702100 #17159] DEBUG -- :   Activity Load (1.7ms)  SELECT "activities".* FROM "activities" WHERE "activities"."kind" = $1 AND "activities"."created_at" BETWEEN $2 AND $3  [["kind", "error"], ["created_at", "2023-01-01"], ["created_at", "2023-01-31"]]

[#<Activity:0x00005589dd710f68
  id: 5589,
  kind: "error",
  browser: "Chrome 108.0.0.0",
  provider_name: "Mira Bobo",
  patient_name: "Mucha, Lucha",
  created_at: Mon, 23 Jan 2023 15:43:07.363213000 PST -08:00,
  updated_at: Mon, 23 Jan 2023 15:43:07.363213000 PST -08:00,
  date: Mon, 23 Jan 2023 15:43:07.426000000 PST -08:00,
  provider_id: "some-uuuid",
  provider_kind: "physician",
  extras:
   {"poc_id"=>"8a866d13-6feb-47ac-9dab-a8fd4663caed",
	"context"=>"sign open task",
	"text"=>"422"},
  dashboard_id: "18a679f6-4f42-4888-8862-f835f873657c",
  provider_email: "email@email.com">,

 #<Activity:0x00005589e1d80458
  id: 5750,
  kind: "error",
  browser: "Microsoft Edge 109.0.1518.70",
  provider_name: "Robert Downey",
  patient_name: "Man, Iron",
  created_at: Mon, 30 Jan 2023 09:36:02.576291000 PST -08:00,
  updated_at: Mon, 30 Jan 2023 09:36:02.576291000 PST -08:00,
  date: Mon, 30 Jan 2023 09:36:02.438000000 PST -08:00,
  provider_id: "some-uuuid",
  provider_kind: "physician",
  extras:
   {"poc_id"=>"f0d1d212-31ee-4070-9f45-1bad4e1f4c52",
	"context"=>"sign open task",
	"text"=>"422"},
  dashboard_id: "8e0e35ac-820b-4fde-82a8-3209c8436a29",
  provider_email: "email@email.com">]
```