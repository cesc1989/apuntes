# ✅ Missing Progress Forms in Summary Page
This [Summary Page](https://forms.getluna.com/patients/67b1e1bc-f94f-40e2-aa83-2055fe90591c/summary/b05f59d1-0e56-441f-a013-09c97ab3f1e6) is reported because it’s missing some of its progress forms. It looks like this as the time of writing.

![](https://paper-attachments.dropbox.com/s_D7A479DFB9D6172FC15621F0F81DB925057E90F71C44D70725D7B8FD05067EA8_1626909563969_Screen+Shot+2021-07-21+at+6.13.31+PM.png)


It all looks fine until we dig in the data. These are the IDs of the progress forms that belong to the form:

    Onb: 7f08a254-e12c-458c-807b-eb72224ddfe4 - "2021-03-17"
    Pro: 707d834a-f521-411b-8cb7-13de67ce6666 - "2021-07-15"
    Pro: d0a80fbf-61e7-483a-bef8-7b11d7cc0033 - "2021-06-07"
    
    Missing: 3944b82e-46b8-4931-8699-a0a95fb5cc40 - incomplete 28484
    Missing: 83388cdf-ec24-417b-9ccf-8323938cecad - incomplete 27218

Let’s dig deeper to try to see where it could be failing.

**Onboarding. Same as 1st outcome measure (from bottom to top)**

    id: 25004,
    patient_id: 8457,
    progress_type: "onboarding",
    type_name: "lefs",
    injury_name: "Hip",
    completed: true,
    completed_at: "2021-03-17",
    uuid: "7f08a254-e12c-458c-807b-eb72224ddfe4",
    form_type_id: 4,
    care_plan_id: "264bba18-50a5-45c4-a637-f272de8fc88b"
## Present Ongoings

**Ongoing #1**

    id: 32429,
    patient_id: 8457,
    progress_type: "ongoing",
    type_name: "lefs",
    injury_name: "Hip",
    completed: true,
    completed_at: "2021-07-15",
    uuid: "707d834a-f521-411b-8cb7-13de67ce6666",
    form_type_id: 4,
    onboarding_id: 25004,
    care_plan_id: "264bba18-50a5-45c4-a637-f272de8fc88b"

**Ongoing #2**

    id: 30727,
    patient_id: 8457,
    progress_type: "ongoing",
    type_name: "lefs",
    injury_name: "Hip",
    completed: true,
    completed_at: "2021-06-07",
    uuid: "d0a80fbf-61e7-483a-bef8-7b11d7cc0033",
    form_type_id: 4,
    onboarding_id: 25004,
    care_plan_id: "264bba18-50a5-45c4-a637-f272de8fc88b"

**Ongoing #3**

    id: 34221,
    patient_id: 8457,
    progress_type: "ongoing",
    type_name: "lefs",
    injury_name: "Hip",
    completed: false,
    uuid: "0434590c-f0b4-4ab9-b846-9a33c90373c8",
    form_type_id: 4,
    onboarding_id: 25004,
    care_plan_id: "264bba18-50a5-45c4-a637-f272de8fc88b",
## Missing Ongoings

**Ongoing #1**

    id: 28484
    patient_id: 8457,
    progress_type: "ongoing",
    type_name: "lefs",
    injury_name: "Hip",
    completed: false,
    uuid: "3944b82e-46b8-4931-8699-a0a95fb5cc40",
    form_type_id: 4,
    onboarding_id: 25004,
    care_plan_id: "264bba18-50a5-45c4-a637-f272de8fc88b"

**Ongoing #2**

    id: 27218
    patient_id: 8457,
    progress_type: "ongoing",
    type_name: "lefs",
    injury_name: "Hip",
    completed: false,
    uuid: "83388cdf-ec24-417b-9ccf-8323938cecad",
    form_type_id: 4,
    onboarding_id: 25004,
    care_plan_id: "264bba18-50a5-45c4-a637-f272de8fc88b"


## Suspicions

**First: forms not linked to onboarding**
Some of the forms might not be “linked” to the onboarding. However, this isn’t true as all forms have the correct `onboarding_id` value: **25004**.

**Second: missing attributes to match**
Similar as first suspicion but with other important attributes. This one is also false as all ongoings have the same values in the important attributes:

- progress_type
- patient_id
- care_plan_id

**Third: interleaving is failing**
With all previous being false, I decided to replicate a similar situation in local env. I took an onboarding and created several progress forms. Then, I proceded to complete them interchangeably in order to create a scenario that could be the cause and could signal a problem in the `SummaryFormInterleaver` class.

This failed as well because it all worked fine:

![](https://paper-attachments.dropbox.com/s_D7A479DFB9D6172FC15621F0F81DB925057E90F71C44D70725D7B8FD05067EA8_1626984179205_image.png)


