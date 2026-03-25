# NEX-3465 - Shoulders for Pain Spots

Etiquetas: #luna_help_desk 

In the Body Chart section of the Intake Form, when the patient indicates shoulder in the back or front chart it selects the other as well. If they choose back shoulder the front gets selected as well.

Screenshot:
![[nex3465.should.error.png]]

## Error 🐞

This happens because in the database, there are only two pain spots records for should sections. This is in contrast to elbow which has four records.

|id|name|created_at|updated_at|
|--|----|----------|----------|
|3|Right Shoulder|2018-08-24 17:52:59.470|2018-10-10 13:51:54.237|
|25|Left Shoulder|2018-10-10 13:52:12.281|2018-10-10 13:52:12.281|
|5|Front Right Elbow|2018-08-24 17:52:59.481|2018-10-10 13:51:54.231|
|26|Front Left Elbow|2018-10-10 13:52:12.285|2018-10-10 13:52:12.285|
|36|Back Right Elbow|2018-10-10 13:52:12.325|2018-10-10 13:52:12.325|
|37|Back Left Elbow|2018-10-10 13:52:12.330|2018-10-10 13:52:12.330|

## Solution

The solution is to create two new more pain spots to identify back parts of the shoulder. To be done in a migration:
```ruby
PatientSelfReport::PainSpot.find(3).update!(name: "Front Right Shoulder")
PatientSelfReport::PainSpot.find(25).update!(name: "Front Left Shoulder")
PatientSelfReport::PainSpot.create!(name: "Back Right Shoulder")
PatientSelfReport::PainSpot.create!(name: "Back Left Shoulder")
```

**Revert:**
```ruby
PatientSelfReport::PainSpot.find(3).update!(name: "Right Shoulder")
PatientSelfReport::PainSpot.find(25).update!(name: "Left Shoulder")
PatientSelfReport::PainSpot.where(name: ["Back Right Shoulder", "Back Left Shoulder"]).destroy_all
```