# [EvW] Notifications Table
This document describes all notifications setup for the Exercises via Web project. Where they generate from, the conditions they have to met in order to be sent, kind of notification.


> Note: patient entity is accessed through the exercise_program’s episode relationship
# Create Exercise Program
## Generated from

Class: *ExerciseProgram*
Method: *check_download_app_email*

## Email Notification
| **Conditions needed**                    | **Expected value**     | **Handled** |
| ---------------------------------------- | ---------------------- | ----------- |
| exercise_program.episode.discharged      | has to be false        | Not needed  |
| patient.download_app_exercise_email_sent | has to be nil or false | ✅           |
| exercise_program.checked_out             | has to be true         | ✅           |
| patient.account.sign_in_count            | has to be zero or nil  | ✅           |

## SMS Notification
| **Conditions needed**               | **Expected value**    | **Handled** |
| ----------------------------------- | --------------------- | ----------- |
| exercise_program.episode.discharged | has to be false       | Not needed  |
| exercise_program.checked_out        | has to be true        | ✅           |
| patient.account.sign_in_count       | has to be zero or nil | ✅           |

# Update Exercise Program AND No Patient App Login
## Generated from

Class: *ExerciseProgram*
Method: *send_update_program_email*

## Email Notification
| **Conditions needed**               | **Expected value**    | **Handled** |
| ----------------------------------- | --------------------- | ----------- |
| exercise_program.episode.discharged | has to be false       | ❌           |
| patient.account.sign_in_count       | has to be zero or nil | ✅           |

# Pending Exercise Workout AND No Patient App Login

Class: *WiltedTreeExercisesViaWebReminderWorker*

## Email Notification
| **Conditions needed**                        | **Expected value**                        | **Handled** |
| -------------------------------------------- | ----------------------------------------- | ----------- |
| exercise_program.episode.discharged          | has to be false                           | ✅           |
| patient.region_id                            | has to be included in regions list        | ✅           |
| patient.account.sign_in_count                | has to be zero or nil                     | ✅           |
| exercise_program.episode                     | has to be equal to patient.recent_episode | ✅           |
| exercise_program.workouts                    | has to have at least one                  | ✅           |
| No recent workout relative to weekly cadence | has to be nil                             | ✅           |

# Wilted tree, Pending Exercise Workout AND No Patient App Login
## Generated from

Class: *WiltedTreeExercisesViaWebReminderWorker*

## Email Notification
| **Conditions needed**                        | **Expected value**                        | **Handled** |
| -------------------------------------------- | ----------------------------------------- | ----------- |
| exercise_program.episode.discharged          | has to be false                           | ✅           |
| patient.region_id                            | has to be included in regions list        | ✅           |
| patient.account.sign_in_count                | has to be zero or nil                     | ✅           |
| exercise_program.episode                     | has to be equal to patient.recent_episode | ✅           |
| exercise_program.workouts                    | has to have at least one                  | ✅           |
| No recent workout relative to weekly cadence | has to be nil                             | ✅           |
| exercise_program.tree_level                  | has to be wilted                          | ✅           |

# Send SMS with link to app from fallback page
## Generated from

Class: *SendAppLinkSmsController*

## SMS Notification

No conditions.

