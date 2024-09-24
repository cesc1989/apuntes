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
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1705677371895_image.png)

## SMS (Jan 19th)
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1705697638526_image.png)

## SMS Shortened URL (Feb 2nd)
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1706889727016_image.png)

# Update Exercise Program AND No Patient App Login

Description: sent after a Therapist updates an assigned exercise program of a patient.
Notification type: Email

## Email **(Jan 19th)**
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1705677747272_image.png)

# Pending Exercise Workout AND No Patient App Login

Notification type: Email

## Email (**Jan 23)**
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1706046926370_image.png)

# Wilted Tree, Pending Exercise Workout AND No Patient App Login

Notification type: Email

## Email (Jan 22nd)
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1705938299466_image.png)

# Send SMS with link to app from fallback page

Tested: January 16th.
Notification type: SMS

## SMS
![](https://paper-attachments.dropboxusercontent.com/s_33CDDFB428CD37AE0F92C8A355E325BE3F02F24AA546000DDD0DC85464321763_1705438502077_image.png)



