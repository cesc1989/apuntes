# Determining whether a Patient should be displayed

How can someone non-eng be able to identify, from different sources, whether a Patient should be displayed in the Seen or Unseen sections?

# Seen Patients

> [!important]
> Context: To show a Patient in the Seen Patients section on any dashboard kind, a Patient should have the `[kind]_id` value that links to the corresponding kind.

~~*(1) Where the Athena value comes from. (e.g., which database). i.e., How does Athena get the data?*~~

*(2) Q will identify where someone can go to see that data. i.e., Where is the best place for a non-Eng to go see that data?*

## Physicians

The identifier is `physician_id`.

This ID is added to Patients records when exported to Athena from Luxe DB.

### How to Check in Luxe?

> [!Note]
> This is taken from a convo in Slack

**Q: Can someone help me understand what's the relationship between `episode.physician` and `episode.referral_physician_name` ?**

A: _sometimes we don’t have the physician in our system in which case the agent just types their name in a free text field. Clinical team follows up later and makes sure the physician is built in HubSpot and attached in the care plan_

## Physician Groups

Apply same setup as for Physicians.

## Clinics

The identifier is `clinic_id`.

This ID is added to Patients records when exported to Athena from Luxe DB.

## Partners

The identifier is `practice_id`.

This ID is added to Patients records when exported to Athena from Luxe DB.

# Unseen Patients

Context: all data shown in the Unseen sections is pulled _only_ from Hubspot.

~~*(1) What is the query looking for to determine which dashboard an unseen patient should be on? what does query say?*~~

~~*(2) Where does someone look in Hubspot to find that value?*~~

## Physicians

> [!important]
> Context: To show Patients in the Unseen section of a Physician Dashboard, Patients Hubspot contacts have to be previously associated through the association label `Referred Patient` to the Physician.

In a Physician dashboard, to display Unseen patients, Patient Hubspot contacts are pulled using the Physician's Hubspot contact ID. These Patient Hubspot contacts are looked for in the list of `Referred Patient`s association label of the Physician.

Any associated patient will show in the Unseen section, except when the Patient Hubspot contact's `hs_lead_status` property value is equals to `Patient - Qualified: Booked in Luxe`.

To check if a Patient will show up in a Physician Dashboard Unseen section, in the Patient's Hubspot profile, under the "Contacts" section on the right sidebar, the physician should appear as `Reffering Physician`:

![[30.referring.physician.patient.hubspot.png]]


## Clinics

> [!important]
> Context: To show Patients in the Unseen section of a Clinic Dashboard, Patients Hubspot contacts' `powered_by_luna_code` needs to be equals the Clinic three chars code.

In a Clinic dashboard, to display Unseen patients, Patients Hubspot contacts need to match the next criteria:

- `lead_source` property equals to `Powered by Luna`
- `powered_by_luna_code` property equals to Clinic three chars code (i.e, MHS for Memorial)
- `hs_lead_status` property to be any of the Lead Status values predefined.

To check if a Patient will show up in a Clinic Dashboard Unseen section, in the Patient's Hubspot profile do the following:

- Click "Actions" button
- Click "View all properties"
- In the search input look for the values for the mentioned properties: `lead_source`, `powered_by_luna_code`, `hs_lead_status`.

## Partners

> [!important]
> Context: To show Patients in the Unseen section of a Partner Dashboard, Patients Hubspot contacts' `powered_by_luna_code` needs to be equals the Partner three chars code.

In a Partner dashboard, to display Unseen patients, Patients Hubspot contacts need to match the next criteria:

- `lead_source` property equals to `Powered by Luna`
- `powered_by_luna_code` property equals to Partner three chars code (i.e, MHS for Memorial)
- `hs_lead_status` property to be any of the Lead Status values predefined.

To check if a Patient will show up in a Partner Dashboard Unseen section, in the Patient's Hubspot profile do the following:

- Click "Actions" button
- Click "View all properties"
- In the search input look for the values for the mentioned properties: `lead_source`, `powered_by_luna_code`, `hs_lead_status`.