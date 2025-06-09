# KX Modifier - Q&A

# What are the conditions for the GraphQL Query

The query is `promptTherapistForMedicalNecessity`. It returns a boolean flag on true when the Care Plan fulfills all conditions in regards of exceeding the spending limite.

These can be found in `EpisodePolicy#prompt_therapist_for_medical_necessity`.

- Therapists has to have access to the care plan
- Care Plan has to be a Medicare one
- A Medicare Dollar Threshold Status (MDTS) exists
- The MDTS's `threshold_exceeded` flag is set as true
- A Medicare Care Plan Medical Necessity Response _does not_ exists
- At least one of the Care Plan's appointments' chart is marked as signed

# Is the `threshold_exceed` flag once per Care Plan?

Yes. It is evaluated per Care Plan. However, in a given year, once the spending limit was exceeded (`threshold_exceed` is true), it will remain true until the next year.