# Mapeo de Secciones y Propiedades de Credentialing Object

El form en Therapist Signup se componen de varias secciones. A continuación mapeo qué propiedades del objeto Credentialing hay que actualizar según la sección en donde se guardan sus valores.

# 1 - Signup

| Field Name               | HS Property Name        | Fuente    |
|--------------------------|-------------------------|-----------|
| Desired Signature        | desired_signature       | therapist |
| BLS Certificate Number   | bls_certificate_number  | therapist |
| BLS Expiration Date      | bls_expiration_date     | therapist |
| BLS Issuing Company      | bls_issuing_company     | therapist |


# 2 - Personal Information

| Field Name          | HS Property Name   | Fuente   |
|---------------------|--------------------|----------|
| Name Change         | name_change        | therapist.aliases.any? |
| Aliases             | aliases            | therapist.aliases |
| Ethnicity           | ethnicity          | therapist |
| Other Ethnicity     | other_ethnicity    | therapist |
| US Citizen          | us_citizen         | therapist |
| Date of Birth       | date_of_birth      | therapist |
| Country of Birth    | country_of_birth   | therapist |
| City of Birth       | city_of_birth      | therapist |
| Place of Birth      | place_of_birth     | therapist |

**Notas**

- Para la propiedad `name_change` se define si el therapist tiene aliases.


# 3 - Immunization

| Field Name                                                            | HS Property Name                                                     |
|-----------------------------------------------------------------------|----------------------------------------------------------------------|
| Covid-19 Vaccination Status                                           | covid_19_vaccination_status                                          |
| Flu Vaccine Status                                                    | flu_vaccine_status                                                   |
| MMR (Measles, Mumps, Rubella) Vaccination Status                      | mmr__measles__mumps__rubella__vaccination_status                     |
| Varicella (Chickenpox) Vaccination Status                             | varicella__chickenpox__vaccination_status                            |
| Hepatitis B Vaccination Status                                        | hepatitis_b_vaccination_status                                       |
| TDAP (Tetanus, Diphtheria, Pertussis) Vaccination Status              | tdap__tetanus__diphtheria__pertussis__vaccination_status             |
| TB Questionnaire: Positive Tuberculosis skin test (PPD)               | tb_questionnaire__positive_tuberculosis_skin_test__ppd_              |
| TB Questionnaire: Current TB Symptoms                                 | tb_questionnaire__current_tb_symptoms                                |
| TB Questionnaire: Positive Tuberculosis blood test (Quantiferon Gold) | tb_questionnaire__positive_tuberculosis_blood_test__quantiferon_gold_|
| TB Questionnaire: Taken medication for Tuberculosis?                  | tb_questionnaire__taken_medication_for_tuberculosis_                 |
| TB Questionnaire: Received BCG vaccine                                | tb_questionnaire__received_bcg_vaccine                               |
| TB Questionnaire: Contact with Individual with TB                     | tb_questionnaire__contact_with_individual_with_tb                    |
| TB Questionnaire: Lived in Countries Outside the US                   | tb_questionnaire__lived_in_countries_outside_the_us                  |
| Immunization and Vaccination Attestation Details                      | immunization_and_vaccination_attestation_details                     |


# 4 - Credentialing Information

| Field Name                                                    | HS Property Name                                                 |
|---------------------------------------------------------------|------------------------------------------------------------------|
| Bachelor's Degree School Start Date                           | bachelor_s_degree_school_start_date                              |
| Bachelor's Degree School Graduation Date                      | bachelor_s_degree_school_graduation_date                         |
| Bachelor's Degree Field of Study                              | bachelor_s_degree_field_of_study                                 |
| Bachelor's Degree Other Field of Study                        | bachelor_s_degree_other_field_of_study                           |
| Bachelor's Degree School Name                                 | bachelor_s_degree_school_name                                    |
| Bachelor's Degree diploma or transcripts or FCCPT Certificate | bachelor_s_degree_diploma_or_transcripts_or_fccpt_certificate    |
| Bachelor's Degree Missing                                     | bachelor_s_degree_missing                                        |
| Master's Degree School Start Date                             | master_s_degree_school_start_date                                |
| Master's Degree School Graduation Date                        | master_s_degree_school_graduation_date                           |
| Master's Degree Field of Study                                | master_s_degree_field_of_study                                   |
| Master's Degree Other Field of Study                          | master_s_degree_other_field_of_study                             |
| Master's Degree School Name                                   | master_s_degree_school_name                                      |
| Master's Degree diploma or transcripts or FCCPT Certificate   | master_s_degree_diploma_or_transcripts_or_fccpt_certificate      |
| Master's Degree Missing                                       | master_s_degree_missing                                          |
| Doctorate Degree School Start Date                            | doctorate_degree_school_start_date                               |
| Doctorate Degree School Graduation Date                       | doctorate_degree_school_graduation_date                          |
| Doctorate Degree Field of Study                               | doctorate_degree_field_of_study                                  |
| Doctorate Degree Other Field of Study                         | doctorate_degree_other_field_of_study                            |
| Doctorate Degree School Name                                  | doctorate_degree_school_name                                     |
| Doctorate Degree diploma or transcripts or FCCPT Certificate  | doctorate_degree_diploma_or_transcripts_or_fccpt_certificate     |
| Doctorate Degree Missing                                      | doctorate_degree_missing                                         |
| Residency                                                     | residency                                                        |
| Residency Institution                                         | residency_institution                                            |
| Residency Start Date                                          | residency_start_date                                             |
| Residency Completion Date                                     | residency_completion_date                                        |
| Fellowship                                                    | fellowship                                                       |
| Fellowship Institution                                        | fellowship_institution                                           |
| Fellowship Start Date                                         | fellowship_start_date                                            |
| Fellowship Completion Date                                    | fellowship_completion_date                                       |
| Board Certifications or Specializations                       | board_certifications_or_specializations                          |
| Board or Specialization Name                                  | board_or_specialization_name                                     |
| Certificate Number                                            | certificate_number                                               |
| Certificate Issue Date                                        | certificate_issue_date                                           |
| Certification Expiration Date                                 | certification_expiration_date                                    |
| Licensed in Other States                                      | licensed_in_other_states                                         |


# 5 - Employment & Malpractice

| Field Name                                      | HS Property Name                                 |
|-------------------------------------------------|--------------------------------------------------|
| Employer Name                                   | employer_name                                    |
| Malpractice Liability Insurance Carrier         | malpractice_liability_insurance_carrier          |
| Malpractice Liability Insurance Policy Number   | malpractice_liability_insurance_policy_number    |
| Malpractice Liability Insurance Expiration      | malpractice_liability_insurance_expiration       |


# 6 - Preferences

| Field Name                                                                 | HS Property Name                                                     |
|----------------------------------------------------------------------------|----------------------------------------------------------------------|
| Desired Weekly Appointments                                                | desired_weekly_appointments                                          |
| Specialties Credentialing Form                                             | specialties_credentialing_form                                       |
| Treatment Styles                                                           | treatment_styles                                                     |
| Therapist requests support on documentation and CPT Codes for outpatient billing | therapist_requests_support_on_documentation_and_cpt_codes_for_outpatient_billing |
| Do you have any pet allergies?                                             | do_you_have_any_pet_allergies_                                       |
| Languages                                                                  | languages                                                            |
| Bio Description                                                            | bio_description                                                      |


# 7 - NPI and CAQH

| Field Name                    | HS Property Name           |
|-------------------------------|----------------------------|
| Existing Therapist NPI        | existing_therapist_npi     |
| Therapist NPI Number          | therapist_npi_number       |
| Medicare Account Status       | medicare_account_status    |
| Existing CAQH                 | existing_caqh              |
| CAQH Number                   | caqh_number                |
| CAQH Username                 | caqh_username              |
| CAQH Password                 | caqh_password              |
| NPI Permission to Luna        | npi_permission_to_luna     |
| CAQH Permission to Luna       | caqh_permission_to_luna    |


# 8 - Medicare Requirement

No aplica.

# 9 - Payouts

No aplica.

# 10 - Certification & Release

Estos nos son campos pero valores "quemados" a partir de la petición que hace el submit del form.

| Field Name                    | HS Property Name         |
|-------------------------------|--------------------------|
| Agreement/Disclaimer          | agreement_disclaimer     |
| Credentialing Form Status     | credentialing_form_status|
