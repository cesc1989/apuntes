# [EvW] Notifications QA Results

Results of testing the [new notifications](https://docs.google.com/spreadsheets/d/1ds5dS3s-TDAyzwxWGz0a4rCr6YOyck5RRHn_CsV3ILE/edit#gid=0).

# Cases and expected notification

| **Event**                                                      | **Type**    | **Regular URL** | **Shortened URL** |
| -------------------------------------------------------------- | ----------- | --------------- | ----------------- |
| Create Exercise Program                                        | Email / SMS | ✅  / ✅          | 🚫 / ✅            |
| Update Exercise Program AND No Patient App Login               | Email       | ✅               | 🚫                |
| Pending Exercise Workout AND No Patient App Login              | Email       | ✅               | 🚫                |
| Wilted tree, Pending Exercise Workout AND No Patient App Login | Email       | ✅               | 🚫                |
| Send SMS with link to app from fallback page                   | SMS         | ✅               | 🚫                |

# Create Exercise Program

Description: sent after a Therapist creates an exercise program for the patient.
Notification type: SMS, Email.

## Email **(Jan 19th)**

![[01.email.test.png]]

## SMS (Jan 19th)

![[02.sms.test.png]]

## SMS Shortened URL (Feb 2nd)

![[03.sms.test.png]]

# Update Exercise Program AND No Patient App Login

Description: sent after a Therapist updates an assigned exercise program of a patient.
Notification type: Email

## Email **(Jan 19th)**

![[04.email.test.png]]

# Pending Exercise Workout AND No Patient App Login

Notification type: Email

## Email (**Jan 23)**

![[05.email.test.png]]

# Wilted Tree, Pending Exercise Workout AND No Patient App Login

Notification type: Email

## Email (Jan 22nd)

![[06.email.test.png]]

# Send SMS with link to app from fallback page

Tested: January 16th.
Notification type: SMS

## SMS

![[07.sms.test.png]]
