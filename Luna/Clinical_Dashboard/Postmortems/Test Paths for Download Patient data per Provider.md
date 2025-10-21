# Regression Test Plan for Patient Data Download in Clinical Dashboard

These are the steps to verify any modification to this flow keeps the same behavior of only downloading the data corresponding to the provider's dashboard.

This feature is available for these providers of the Clinical Dashboard:

- Physicians
- Clinics
- Practices

The objective of this regression test is to make sure the requested download data is scoped correctly only to the provider in the Clinical Dashboard.

## Regression Testing in Alpha

1. Generate a new provider link with the following curl request.

Considerations:
- Indicate an email address you use AND it's verified in the provider recipients list in Luxe

Use this curl request:
```bash
curl --request POST \
  --url https://api-provider-portal.alpha.getluna.com/v2/links/ \
  --header 'authorization: Token eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJQbGVKTzNvRWJLZWxHZ29TUUlOTGdxckhZRVIyUGpwayJ9.FPIyjLV6VK7zCDrmNfusX7_cem3zOdBeFe1IFTMAeOk' \
  --header 'content-type: application/json' \
  --data '{
  "link_recipients": [
    {
      "provider_name": "duke_health_greensboro",
      "provider_kind": "clinic",
      "provider_id": "ca118f55-38cd-4b92-9d28-691947ac9d03",
      "portal_recipient_email": "francisco.quintero+101@ideaware.co"
    }
  ]
}'
```

2. Copy the generated URL and open it in a web browser.
3. Navigate down to the "All Seen Patients" section.
	1. If you don't see patient data try choosing "All Time" from the "Select Time Period" in the top of the page.
4. Click the "Download All" button in the bottom right corner of this section.
5. Check your inbox for a download link and click it to download the CSV file.
6. Open the CSV file and check the data of patients matches what is shown in the "All Seen Patients" in the dashboard.
	1. Look for the same amount of patients.

## Code

These are the files covering this functionality. Although these all live in `backend` repository the functionality is provided by two different sub-systems:

- data generation: Clinical Dashboard
- download URL email: Edge

### Clinical Dashboard

In order of execution:

- Controller: `app/controllers/clinical_dashboard/api/v1/download_patient_data_controller.rb`
- Worker: `app/workers/clinical_dashboard/download_patients_csv_worker.rb`
- Data generation: `app/lib/clinical_dashboard/download_patient_data.rb`
- Luxe API call: `app/services/clinical_dashboard/email_patients_csv_service.rb`

### Luxe

Controller: `app/controllers/api/v1/internal/provider_communications_controller.rb`
Method: `send_patients_csv_download_link`