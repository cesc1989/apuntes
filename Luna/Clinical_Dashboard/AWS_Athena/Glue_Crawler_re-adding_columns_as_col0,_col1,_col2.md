# Glue Crawler re-adding columns as col0, col1, col2, etc

For the appointments (thumbs up/down) export, the crawler does not add the correct column names instead it creates them as col0, col1, col2, etc. Why does this happen?

Possible causes and [solutions in here](https://stackoverflow.com/questions/61857098/aws-glue-not-detecting-header-in-csv).

In the [“Built-in CSV classifier” subsection](https://docs.aws.amazon.com/glue/latest/dg/add-classifier.html) we can find more info about how they work and why this could happen.

# Setup

This is a sample of the csv export:
```csv
care_plan_id|patient_id|therapist_name|appointment_id|created_at|scheduled_at|visit_type|status|patient_rating
    6c0b024d-b24c-4ece-a9a6-00839ba28b94|35b8aa82-a3e6-4c16-9dc0-0026d8fe658e|Richard Eaden|9ff1239d-6abe-4caa-87c6-aff0a0229620|2022-10-21T12:46:47.451908|2022-10-28T15:45:00|standard|completed|
```

This is how the crawler creates the table in Glue:
![[created.table.crawler.png]]

These are the “Advanced Properties”. I had to manually add the `skip.header.line.count` one.

![[glue.table.properties.png]]

And this is how the column names are being created:

![[bad.column.names.png]]

These are the correct column names after I manually fix them. However, the crawler runs again, they’re changed to the “col0, col1” version.

![[glue.corrected.columns.png]]

# Classifier

The classifier the crawler uses to read data from source and recreate tables is setup:

![[glue.classifier.png]]

# Possible Causes

In the docs it says:

> The header row must be sufficiently different from the data rows. To determine this, one or more of the rows must parse as other than STRING type. If all columns are of type STRING, then the first row of data is not sufficiently different from subsequent rows to be used as the header.

All values being exported and the column headers are of type string, this could be the reason the columns aren’t named as expected. Let’s compare them with other tables.

- Appointments table: all columns as type string
- Partner clinics table: has three columns of type boolean
- Physicians table: has three columns of type boolean
- Patients table: has some columns of type double, boolean, bigint, etc
- Physician groups table: has three columns of type boolean

Then it could be that having all columns as string is causing the issue with the crawler.

