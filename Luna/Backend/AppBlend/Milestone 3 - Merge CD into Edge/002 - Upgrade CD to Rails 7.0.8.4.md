# Upgrade Clinical Dashboard to Rails 7.0.8.4

# Download CSV Data to Email

Everything looks fine but the email with the CSV of Recent Patients data does not arrive.

The whole flow looks good but no email in inbox.

## Manual request to send email from Dashboard

Logged in to dashboard container and run this in a rails console:
```ruby
EmailPatientsCsvService.new(
  provider_id: nil,
  provider_kind: nil,
  email: "francisco.quintero@ideaware.co",
  url: "https://luna-alpha-workloads-patient-csv-downloads.s3.us-west-2.amazonaws.com/francisco.quintero%40ideaware.co_2025_04_02_data.csv",
  is_admin: "true"
).deliver
```

In the Graphana logs for Edge it can be seen the response for the endpoint is a 204 (expected).

Query:
```
{app="edge"} |= `send_patients_csv_download_link`
```

Results:
```
2025-04-02 16:20:24.246	
I, [2025-04-02T21:20:24.246090 #281]  INFO -- : [fd410d5ef60dec42fcf348ed6c2a59f8] [0] method=POST path=/api/v1/internal/provider_communications/send_patients_csv_download_link format=html controller=Api::V1::Internal::ProviderCommunicationsController action=send_patients_csv_download_link status=204 allocations=1180 duration=3.24 view=0.00 db=0.00

I, [2025-04-02T21:15:08.988130 #20]  INFO -- : [8bc51f12fd8b5dc878b87c7d3dde6b45] [0] method=POST path=/api/v1/internal/provider_communications/send_patients_csv_download_link format=html controller=Api::V1::Internal::ProviderCommunicationsController action=send_patients_csv_download_link status=204 allocations=1184 duration=3.43 view=0.00 db=0.00

I, [2025-04-02T20:50:41.949112 #22]  INFO -- : [4bbfbac9bf20470785dd06e7fb4dc99d] [0] method=POST path=/api/v1/internal/provider_communications/send_patients_csv_download_link format=html controller=Api::V1::Internal::ProviderCommunicationsController action=send_patients_csv_download_link status=204 allocations=1033 duration=4.10 view=0.00 db=0.00

I, [2025-04-02T20:50:30.613866 #22]  INFO -- : [6236273b7e086d87e69deb6838dc4d82] [0] method=POST path=/api/v1/internal/provider_communications/send_patients_csv_download_link format=html controller=Api::V1::Internal::ProviderCommunicationsController action=send_patients_csv_download_link status=204 allocations=1182 duration=4.10 view=0.00 db=0.00

I, [2025-04-02T20:49:03.813774 #224]  INFO -- : [6678759f89c2d52d1bed2455ba2c354d] [0] method=POST path=/api/v1/internal/provider_communications/send_patients_csv_download_link format=html controller=Api::V1::Internal::ProviderCommunicationsController action=send_patients_csv_download_link status=204 allocations=1180 duration=3.65 view=0.00 db=0.00
```

## Manual send email from Edge

Hit this manually from a rails console in an Edge container:
```ruby
DashboardAdminMailer.patients_csv_download_link(
  "https://luna-alpha-workloads-patient-csv-downloads.s3.us-west-2.amazonaws.com/francisco.quintero%40ideaware.co_2025_04_02_data.csv",
  "francisco.quintero@ideaware.co"
).deliver_later
```

Got this:
```
I, [2025-04-02T21:29:46.697079 #178]  INFO -- : [ActiveJob] Enqueued ActionMailer::MailDeliveryJob (Job ID: fd8f0a19-ef8e-4875-8cd6-de23389c527a) to Sidekiq(mailers) with arguments: "DashboardAdminMailer", "pa
tients_csv_download_link", "deliver_now", {:args=>["https://luna-alpha-workloads-patient-csv-downloads.s3.us-west-2.amazonaws.com/francisco.quintero%40ideaware.co_2025_04_02_data.csv(...)", "francisco.quintero@ideaware.co"]}
```

```ruby
#<ActionMailer::MailDeliveryJob:0x00007f92350dc720
 @_halted_callback_hook_called=nil,
 @_scheduled_at_time=nil,
 @arguments=
  ["DashboardAdminMailer",
   "patients_csv_download_link",
   "deliver_now",
   {:args=>
     ["https://luna-alpha-workloads-patient-csv-downloads.s3.us-west-2.amazonaws.com/francisco.quintero%40ideaware.co_2025_04_02_data.csv(...)",
      "francisco.quintero@ideaware.co"]}],
 @exception_executions={},
 @executions=0,
 @job_id="fd8f0a19-ef8e-4875-8cd6-de23389c527a",
 @priority=nil,
 @provider_job_id="b13a8b58646f2e050ce58e18",
 @queue_name="mailers",
 @scheduled_at=nil,
 @successfully_enqueued=true,
 @timezone="Pacific Time (US & Canada)">
```

So it's all fine. It's more of an Edge issue in Alpha.

## The problem is the mailers queue in alpha

It's stuck with lots of jobs to complete:

![[01.download.data.email.in.queue.png]]