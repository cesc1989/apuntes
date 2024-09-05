# Data Model Explained - Therapists Credentialing

![[Therapists.png]]

This data model is a representation of the sections that define the therapists credentialing form.

Each section is represented by a table with the exception of **Personal Information** and **Certification and Release** which are grouped into *Therapist* table only as we see those fields as something that belongs/represents a therapist.

| **Form Section**                              | **Database Table**         | **Active Record Model**  |
| --------------------------------------------- | -------------------------- | ------------------------ |
| Signup (not shown in design but initial step) | therapists                 | Therapist                |
| Personal Information                          | therapists                 | Therapist                |
| Information for Credentialing                 | credentialing_informations | CredentialingInformation |
| Personal References (subsection)              | personal_references        | PersonalReference        |
| Interest and Preferences                      | preferences                | Preference               |
| NPI and CAQH Application                      | npi_and_caqh_applications  | NpiAndCaqhApplication    |
| Medicare and Private Insurance Requirement    | medicare_requirement       | MedicareRequirement      |
| Details for Earnings Payouts                  | payouts                    | Payout                   |
| Certification and Release                     | therapists                 | Therapist                |

By doing this model we can keep separate form sections into their own tables that would have their own `ActiveRecord` models with validations, methods, and more.

It also would simplify saving and retrieving data for every section as the incoming/outgoing payload would be smaller than if all sections belonged to a same table.

## Questions and Answers Tables

To express and simplify the process of collecting information asked by radio button and checkboxes fields like:


- Are you joining Luna for supplemental income, or as your main source of income? (in the *Details for Earning Payouts* section)
- Treatment styles you feel competent in (in the *Interest and Preference* table)

They’re questions that aren’t something like a Yes/No which could be defined as a *boolean* field in the corresponding table.

By modeling a questions and answers table we try to make this implementation easier. It would be in the sense that we could provide something using `ActiveRecord` like:


    Therapist.first.npi_and_caqh_application.questions

To get the list of questions that would show up in the **NPI and CAQH Application** page. This way, we can also do:


    Therapist.first.npi_and_caqh_application.answers

And should get the amount of answers corresponding to the amount of questions.

