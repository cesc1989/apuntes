# [EvW] Notifications Logic && Files

# What is the logic for the push notification “Pending Exercise Workout”?

There’s multiple cases. Can be seen at `config/locales/en.yml` under the `workouts` key:

    reminder: "Here’s a friendly reminder to get your workout started!"
    
    create_program: "Your therapist has created your exercise plan. Please take a look!"
    
    update_program: "Your therapist has updated your exercise plan. Please take a look!"
    
    wilted_tree: "Oh no! Your tree has wilted as it's been %{day_number} days since your last workout. Complete a workout to make your tree healthy again!"


## [Push] Here’s a friendly reminder to get your workout started!

This push is sent daily, at 7:55am (patient timezone).


> NOTE: see these files to follow the logic: PeriodicJobs (line 282), WorkoutsReminderWorker, and **WorkoutNotificationWorker**

It's sent to patients that:

- are not discharged
- have logins
- do not have recent workouts (recent depends on the cadence of the workout)


## [Push] Oh no! Your tree has wilted as it's been [Number] days since your last workout. Complete a workout to make your tree healthy again!

This push is sent daily, at noon (12pm, patient timezone). 


> NOTE: see these files to follow the logic: PeriodicJobs (line 255), LocalWiltedTreeRemindersWorker, and **WiltedTreeReminderWorker**

It's sent to patients that:

- are not discharged
- have logins
- have previously done exercises
- do not have recent workouts (recent depends on the cadence of the workout)


# What is the logic for Email notification “Wilted Tree, Pending Exercise Workout AND No Patient App Login” && “Pending Exercise Workout AND No Patient App Login”
> NOTE: see these files to follow the logic: PeriodicJobs (line 285), LocalWiltedTreeExercisesViaWebRemindersWorker, and **WiltedTreeExercisesViaWebReminderWorker**

These two distinct email notifications originate from the same source: `WiltedTreeExercisesViaWebReminderWorker`.

Either of the emails is sent to patients that:

- are not discharged
- is in the region the worker is running for
- do not have logins in the mobile app
- have previously done exercises
- do not have workouts in the last 8 days 

Finally, depending on their tree status.

## Wilted Tree, Pending Exercise Workout AND No Patient App Login

If the tree is wilted, they get this notification.

Subject: "Your Tree has Wilted! Reminder to Exercise!"
Body:

    <p>Hi <%= @patient.first_name %>,</p>
    
    <p>Oh no! Your tree has wilted as it's been <%= @absent_days %> days since your last workout.</p>
    
    <p>Log into the Luna App or visit <%= link_to "Luna Exercises on the web", @exercises_web_link %> to complete your workout and make your tree healthy again!</p>
    
    <p>Thank you for using Luna.</p>
    


## Pending Exercise Workout AND No Patient App Login

If the tree is other than wilted, they get this notification.

Subject: "Reminder to Exercise!"
Body:

    <p>Hi <%= @patient.first_name %>,</p>
    
    <p>Here's a friendly reminder to get your workout started! Log into the Luna App or visit <%= link_to "Luna Exercises on the web", @exercises_web_link %> to complete your workout!</p>
    
    <p>Thank you for using Luna.</p>
    

