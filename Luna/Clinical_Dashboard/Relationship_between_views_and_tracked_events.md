# Relationship between views and tracked events

Show the views that trigger events/metrics and the JSON sent to the backend or the data being saved. I don’t want to keep this in my head anymore 🤣 

# Tracked Events

1. Opening charts modal [Patients table]
2. Opening sessions modal [Patients table]
3. Requesting patient data download [Patients table]
4. Signing POC [Open tasks]
5. Opening POC [Open tasks]
6. Opening chart [Charts modal in Patients table]
7. Logging in [Using portal link]
8. Number of unsigned POCs [Open tasks]
9. Requesting new portal link [Clicking request button]
10. General errors [Error wherever they happen]

# In Detail

## 1) Opening Charts Modal

Key: *opened_charts_modal*
String in database: *opened_charts_modal*

Triggered at

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675195576900_Screen+Shot+2023-01-31+at+3.03.30+PM.png)


JSON payload:

```json
{
  "data": {
	"dashboard_id": "d5534a91-c089-473a-8703-cfb379e702a8",
	"provider_email": "francisco.quintero+2@ideaware.co",
	"browser": "Chrome 109.0.0.0",
	"date": "2023-01-31T20:03:22.879Z",
	"provider_id": "some-uudi",
	"provider_name": "Aaron Juancho",
	"provider_kind": "physician",
	"kind": "opened_charts_modal",
	"patient_name": "Mallie Moen",
	"extras": {
	  "patient_id": "fa051e8a-af99-4bbd-a8f8-baf482412d2c"
	}
  }
}
```



## 2) Opening Sessions Modal

Key: *opened_sessions_modal*
String in database: *opened_sessions_modal*

Triggered at

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675195721560_02.sessions.modal.png)


JSON payload:

```json
  {
      "data": {
        "dashboard_id": "d5534a91-c089-473a-8703-cfb379e702a8",
        "provider_email": "francisco.quintero+2@ideaware.co",
        "browser": "Chrome 109.0.0.0",
        "date": "2023-01-31T20:06:53.049Z",
        "provider_id": "some-uuid",
        "provider_name": "Aaron Fuancho",
        "provider_kind": "physician",
        "kind": "opened_sessions_modal",
        "patient_name": "Mallie Moen",
        "extras": {
          "patient_id": "fa051e8a-af99-4bbd-a8f8-baf482412d2c"
        }
      }
    }
```


## 3) Requesting patient data download

Key: *requested_patient_data_download*
String in database: *requested_patient_data_download*

Triggered at

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675196657637_03.request.png)


JSON Payload

```json
  {
        "data":
        {
            "dashboard_id": "d5534a91-c089-473a-8703-cfb379e702a8",
            "provider_email": "francisco.quintero+2@ideaware.co",
            "browser": "Chrome 109.0.0.0",
            "date": "2023-01-31T20:21:32.449Z",
            "provider_id": "some-uuid",
            "provider_name": "Aaron fuancho",
            "provider_kind": "physician",
            "kind": "requested_patient_data_download",
            "patient_name": "Does not apply"
        }
    }
```


## 4) Signing Plan Of Care

Key: *signed_poc*
String in database: *signed_poc*

Triggered at

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675196832044_04.signing.poc.png)


JSON Payload

```json
  {
        "data":
        {
            "dashboard_id": "d5534a91-c089-473a-8703-cfb379e702a8",
            "provider_email": "francisco.quintero+2@ideaware.co",
            "browser": "Chrome 109.0.0.0",
            "date": "2023-01-31T20:26:04.883Z",
            "provider_id": "some-uuid",
            "provider_name": "Aaron Fuancho",
            "provider_kind": "physician",
            "kind": "signed_poc",
            "patient_name": "Stanton, Wilfrid",
            "extras":
            {
                "poc_id": "6ab5e843-9bb8-4edc-b96a-300adebae23d"
            }
        }
    }
```


## 5) Opening Plan Of Care

Key: *opened_plan_of_care*
String in database: *plan_of_care_click*

Triggered at

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675195983111_Screen+Shot+2023-01-31+at+3.11.48+PM.png)

JSON Payload

```json
   {
      "data": {
        "dashboard_id": "d5534a91-c089-473a-8703-cfb379e702a8",
        "provider_email": "francisco.quintero+2@ideaware.co",
        "browser": "Chrome 109.0.0.0",
        "date": "2023-01-31T15:58:02.151Z",
        "provider_id": "some-uuuid",
        "provider_name": "Aaron Fuancho",
        "provider_kind": "physician",
        "kind": "plan_of_care_click",
        "patient_name": "Stanton, Wilfrid",
        "extras": {
          "poc_id": "6ab5e843-9bb8-4edc-b96a-300adebae23d"
        }
      }
    }
```


## 6) Opening Chart

Key: *opened_chart*
String in database: *patient_chart*

Triggered at

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675196467177_opened.chart.png)


JSON Payload

```json
  {
      "data": {
        "dashboard_id": "d5534a91-c089-473a-8703-cfb379e702a8",
        "provider_email": "francisco.quintero+2@ideaware.co",
        "browser": "Chrome 109.0.0.0",
        "date": "2023-01-31T15:54:47.915Z",
        "provider_id": "some-uuid",
        "provider_name": "Aaron Fuancho",
        "provider_kind": "physician",
        "kind": "patient_chart",
        "patient_name": "Mallie Moen",
        "extras": {
          "patient_id": "fa051e8a-af99-4bbd-a8f8-baf482412d2c",
          "chart_id": 9090,
          "care_plan_id": "f95ac685-0315-45a3-a16b-52f80f770170"
        }
      }
    }
```


## 7) Logging in

Key: *logged_in*
String in database: *logged_in*

Triggered at: *clicking portal link in received email*.

Payload

```ruby
kind: EVENTS[:logged_in],
date: DateTime.current,
browser: DOES_NOT_APPLY_TEXT,
provider_name: args['provider_name'],
provider_id: args['provider_id'],
provider_kind: args['provider_kind'],
patient_name: DOES_NOT_APPLY_TEXT,
dashboard_id: args['dashboard_id'],
provider_email: args['portal_recipient_email']
```


## 8) Number of unsigned plans of care

Key: *unsigned_pocs_count*
String in database: *unsigned_pocs_count*

Triggered at: *After requesting pending plans of care from Edge*.

Payload

```ruby
kind: EVENTS[:unsigned_pocs_count],
date: DateTime.current,
browser: DOES_NOT_APPLY_TEXT,
provider_name: args['provider_name'],
provider_id: args['provider_id'],
provider_kind: args['provider_kind'],
patient_name: DOES_NOT_APPLY_TEXT,
extras: { unsigned_pocs_count: args['unsigned_pocs_count'] },
dashboard_id: args['dashboard_id'],
provider_email: args['portal_recipient_email']
```


## 9) Requesting new portal link

Key: *requested_new_link*
String in database: *requested_new_link*

Triggered at:

![](https://paper-attachments.dropboxusercontent.com/s_DC5F13FAACF6782A96CC094C90E74E372559962D2D58F9B9215990D5069D54EC_1675197476130_Screen+Shot+2023-01-31+at+3.37.23+PM.png)


Payload

```ruby
kind: EVENTS[:requested_new_link],
date: DateTime.current,
browser: DOES_NOT_APPLY_TEXT,
provider_name: args['provider_name'],
provider_id: args['provider_id'],
provider_kind: args['provider_kind'],
patient_name: DOES_NOT_APPLY_TEXT,
dashboard_id: args['dashboard_id'],
provider_email: args['portal_recipient_email'] || args['provider_email']
```


## 10) General errors

Key: *general_error*
String in database: *error*

Triggered at: *whenever an error happens*.

Payload

```ruby
kind: EVENTS[:general_error],
date: DateTime.current,
browser: DOES_NOT_APPLY_TEXT,
provider_name: args['provider_name'],
provider_id: args['provider_id'],
provider_kind: args['provider_kind'],
patient_name: DOES_NOT_APPLY_TEXT,
extras: args['extras'],
dashboard_id: args['dashboard_id'],
provider_email: args['portal_recipient_email']
```

