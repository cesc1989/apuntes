# Determining whether a Patient should be displayed

Who can someone non-eng be able to identify, from different sources, whether a Patient should be displayed in the Seen or Unseen Patients tables?


# Seen Patients

Context: To show a Patient in any dashboard kind, a Patient should have the `_id` value that links to the expected kind.

*(1) Where the Athena value comes from. (e.g., which database). i.e., How does Athena get the data?*

## Physician Dashboard

The identifier is `physician_id`.

This ID is added to Patients records when exported from Luxe DB.

## Physician Group

Apply same setup as for Physicians.

## Clinics

The identifier is `clinic_id`.

This ID is added to Patients records when exported from Luxe DB.

## Practices/Partners

The identifier is `practice_id`.

This ID is added to Patients records when exported from Luxe DB.

*(2) Q will identify where someone can go to see that data. i.e., Where is the best place for a non-Eng to go see that data?*

# Unseen Patients

(1) What is the query looking for to determine which dashboard an unseen patient should be on? what does query say?

(2) Where does someone look in Hubspot to find that value?