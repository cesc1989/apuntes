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
  url: "https://luna-alpha-workloads-patient-csv-downloads.s3.us-west-2.amazonaws.com/francisco.quintero%40ideaware.co_2025_04_02_data.csv?response-content-disposition=inline&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEHUaCXVzLXdlc3QtMiJHMEUCIFYNk7wpO4xUSNSNOassySLK8KUMdZGcWWfuDcWD165MAiEAs5fW4UFe8VQCp5JeTLRe%2BVR960KQ20sSTBzUHFc4S9oqpQUI3v%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARAAGgw1MzYyMTg5NjM2NTkiDL3x5xJqIEOKP%2BLUzyr5BCb1fXb6DJyVe3KYtt4RoDUA9u2Js8RRTxblh2HZwZUlep%2Bhc4GdHGMYAf0QSzkszfYw0rxgpbImeGw%2BPBANqdjpojmoUHT9sYL7JtfaKh8KTFdkgRElxjmZi%2BUZ3%2BMZ3Yxk7mh403QWxsgPzA6uR5tsK23i8mbsStWXmYMCaQRShtw7wpZdBxJAebflMdM3AMhyK4a96pMLdfo2Mx5a2XrRkCaaOl8xbwPM%2BQCYj3gkE6SpZq309jzWYpl0nVHDdG1Df%2FdYDFUJTjMF1VvKb0WQjx8nYc0RyLJKvFsMUdXxLNfuOm0LSROv74SiH7xeQGjzOKGQeJR973Fk4b8qeETjoVMiqqF2fgx72S69YYVhkmLxl72VjZvlvrQO%2Bu7o6rHtFogkkKo0HXUBEcRvs%2BCQ7tPrRPB89mPKNrdI8EmSWwE5sibS%2Fblss5hJiXPJXjCK1LDVIMjb8pN5IiF%2BtLxSH4H5vAzAn4yjpKrYAb2LiY%2FvNlnvgZiiK0AiUoczNc1c7wSw1OqEnZFXVHdDOi5S3frEx16SMQrRH3lf3TuBggNn9k6Z2GFxGbR09dxQd7bg5xDI6cl6EHXZhrbvA9AWB0QPQ92o6QT4SL9aM1gofkaaUJWvTXkvSffHQ5fv33y4nXPJtsCPYKOHTUKYKiAwQdTlT5Kr%2Fwy3kmMyo9I%2FW2p8qaeEsj3oEZFcBnG5QUyzH7il%2FQfHdr7VWlKJPJmuMztsGTqSdaeQYld7IZh9IdVHLU0kJ13qQRKtQvH4wskOMLnSMT3unw4PvkQvi4401%2FWNE1SOB4xuWgR7nkkgvVofQFjA8IzBgBU3A4IWXLTpoGx9CkbIkjCtzLa%2FBjrFApZq8ZeYcOFZq9NOmlRGhef6hFf8iKXpuseVcguFuE9bWhgU1vJAOHWD4v7JCG0DwJYIjuKJScJg17tm47e%2FQR78s%2BXHiTPMkMIOTmN9B8%2FDD6ZBjDk4PaaZdOyebVhP9Rfyh8tcHRkzl2CZdZDngYLiqKAXxisdnptFTW0Jsmm7RhcHwmf7SKXgPy%2B6avIw9KLEan2EadE8mvVnXkkdx4NUnwxuw5OGOCxnWWhjNx%2F5AhAlkTGB2CTsSEW3AXko8GiEYylEiy5Voy2uUi9CD5BHCJuyKhrJhoboOdfa0lq4uvBHzQT71JFx5%2B6YovcrMFfKd%2BKj4eLVjnaxPFXTom%2Bqy8xy%2FfM2I1vXlhRpd4FMDx6IvUwiHmMhvp0lKJeDPRc0%2FGeutxZ2Ks6iYF1qkLOYso0C8mvOljljosMq1c7nAjL2tdg%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAXZWJEB3FUTWR3ANO%2F20250402%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20250402T211424Z&X-Amz-Expires=7200&X-Amz-SignedHeaders=host&X-Amz-Signature=35a140c502b99b382d2bde0a7686a4a5219e72eed112df92b1fa449807d6f273",
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
  "https://luna-alpha-workloads-patient-csv-downloads.s3.us-west-2.amazonaws.com/francisco.quintero%40ideaware.co_2025_04_02_data.csv?response-content-disposition=inline&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEHUaCXVzLXdlc3QtMiJHMEUCIFYNk7wpO4xUSNSNOassySLK8KUMdZGcWWfuDcWD165MAiEAs5fW4UFe8VQCp5JeTLRe%2BVR960KQ20sSTBzUHFc4S9oqpQUI3v%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARAAGgw1MzYyMTg5NjM2NTkiDL3x5xJqIEOKP%2BLUzyr5BCb1fXb6DJyVe3KYtt4RoDUA9u2Js8RRTxblh2HZwZUlep%2Bhc4GdHGMYAf0QSzkszfYw0rxgpbImeGw%2BPBANqdjpojmoUHT9sYL7JtfaKh8KTFdkgRElxjmZi%2BUZ3%2BMZ3Yxk7mh403QWxsgPzA6uR5tsK23i8mbsStWXmYMCaQRShtw7wpZdBxJAebflMdM3AMhyK4a96pMLdfo2Mx5a2XrRkCaaOl8xbwPM%2BQCYj3gkE6SpZq309jzWYpl0nVHDdG1Df%2FdYDFUJTjMF1VvKb0WQjx8nYc0RyLJKvFsMUdXxLNfuOm0LSROv74SiH7xeQGjzOKGQeJR973Fk4b8qeETjoVMiqqF2fgx72S69YYVhkmLxl72VjZvlvrQO%2Bu7o6rHtFogkkKo0HXUBEcRvs%2BCQ7tPrRPB89mPKNrdI8EmSWwE5sibS%2Fblss5hJiXPJXjCK1LDVIMjb8pN5IiF%2BtLxSH4H5vAzAn4yjpKrYAb2LiY%2FvNlnvgZiiK0AiUoczNc1c7wSw1OqEnZFXVHdDOi5S3frEx16SMQrRH3lf3TuBggNn9k6Z2GFxGbR09dxQd7bg5xDI6cl6EHXZhrbvA9AWB0QPQ92o6QT4SL9aM1gofkaaUJWvTXkvSffHQ5fv33y4nXPJtsCPYKOHTUKYKiAwQdTlT5Kr%2Fwy3kmMyo9I%2FW2p8qaeEsj3oEZFcBnG5QUyzH7il%2FQfHdr7VWlKJPJmuMztsGTqSdaeQYld7IZh9IdVHLU0kJ13qQRKtQvH4wskOMLnSMT3unw4PvkQvi4401%2FWNE1SOB4xuWgR7nkkgvVofQFjA8IzBgBU3A4IWXLTpoGx9CkbIkjCtzLa%2FBjrFApZq8ZeYcOFZq9NOmlRGhef6hFf8iKXpuseVcguFuE9bWhgU1vJAOHWD4v7JCG0DwJYIjuKJScJg17tm47e%2FQR78s%2BXHiTPMkMIOTmN9B8%2FDD6ZBjDk4PaaZdOyebVhP9Rfyh8tcHRkzl2CZdZDngYLiqKAXxisdnptFTW0Jsmm7RhcHwmf7SKXgPy%2B6avIw9KLEan2EadE8mvVnXkkdx4NUnwxuw5OGOCxnWWhjNx%2F5AhAlkTGB2CTsSEW3AXko8GiEYylEiy5Voy2uUi9CD5BHCJuyKhrJhoboOdfa0lq4uvBHzQT71JFx5%2B6YovcrMFfKd%2BKj4eLVjnaxPFXTom%2Bqy8xy%2FfM2I1vXlhRpd4FMDx6IvUwiHmMhvp0lKJeDPRc0%2FGeutxZ2Ks6iYF1qkLOYso0C8mvOljljosMq1c7nAjL2tdg%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAXZWJEB3FUTWR3ANO%2F20250402%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20250402T211424Z&X-Amz-Expires=7200&X-Amz-SignedHeaders=host&X-Amz-Signature=35a140c502b99b382d2bde0a7686a4a5219e72eed112df92b1fa449807d6f273",
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