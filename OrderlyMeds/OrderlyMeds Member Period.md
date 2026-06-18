# ¿Qué es el Member Period?

> A Member Period is the complete process a patient goes through when ordering more medication in our platform. It starts with the check-in flow, continues through provider review (MSO), prescription fulfillment, and ends with the medication being shipped to the patient.


## Member Period Dates

- **StartDate:** The date we expect the patient to start taking their medicine. Defined as time of pharmacy order confirmation + 3 days.
- **EndDate:** The date we expect the patient to run out of medication. Defined as StartDate + WeeksOfSupply Weeks.
- **NextCheckinEligibleDate:** The earliest date we allow the patient to check back in (therefore the date we create the next MemberPeriod). Defined as EndDate - 15 days.
- **CheckinDueDate:** The date the patient is actually supposed to check in. Defined as the previous Member Period's EndDate.
- **CheckinDeadlineDate:** The date after which the Member Period is canceled and the patient loses their loyalty points. Defined as CheckinDueDate + 15 days.