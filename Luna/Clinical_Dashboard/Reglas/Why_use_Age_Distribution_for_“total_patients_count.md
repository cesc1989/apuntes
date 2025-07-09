# Why use Age Distribution for “total patients count”?

![[total.patients.png]]

Why is the total patients count  from “Age Distribution” graphic is used to fill the text *“Includes data for X patients in last TIME PERIOD”*

Why is not this number taken from the “Recent Patients” table?

There are two reasons:

1. In Patients table we filter patients with the 0/0 case
    1. 0 completed sessions
    2. 0 pending sessions
2. In Patients table we mix data from Patient Forms
    1. This means we can get way more or less data to make a patients count

The data to create the “Age Distribution” graphic comes directly from Luxe. It’s a proper source of truth.

![[age.distribution.png]]

In the other hand, the data to generate the “Recent Patients” table is mixed and filter out to make it more appealing to providers.

![[patients.table.png]]

The screenshots are taken from of Dr. Nick’s dashboard in Alpha environment.

We can see in “Recent Patients” there are only two but in “Age Distribution” the graph places data in multiple bars.

