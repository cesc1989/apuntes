# Campos Date y DateTime que se envían a HubSpot

Para saber qué campos necesitan usar la función que convierte el valor UTC a `in_time_zone`. Esto con respecto a las funciones `formated_date` y `timezone_aware_formated_date`.

## Date Fields (`t.date`) - Use `formated_date`

These fields store date-only values without time components, so no timezone conversion issues:

1. **`@therapist.care_start_date`** - `t.date` - User-selected care start date
2. **`@therapist.physical_therapy_license_expiration_date`** - `t.date` - License expiration date
3. **`@therapist.basic_life_support_expiration_date`** - `t.date` - BLS certificate expiration
4. **`@credentialing.physical_therapy_license_initial_date`** - `t.date` - License initial issue date
5. **`@credentialing.direct_access_expiration`** - `t.date` - Direct access license expiration
6. **`@credentialing.residency_program_start_date`** - `t.date` - Residency program start
7. **`@credentialing.residency_program_completion_date`** - `t.date` - Residency program completion
8. **`@credentialing.fellowship_program_start_date`** - `t.date` - Fellowship program start
9. **`@credentialing.fellowship_program_completion_date`** - `t.date` - Fellowship program completion
10. **`@credentialing.board_certification_date`** - `t.date` - Board certification issue date
11. **`@credentialing.board_certification_expiration_date`** - `t.date` - Board certification expiration
12. **`highest_education_info[:start_date]`** - `t.date` - Education start date
13. **`highest_education_info[:graduation_date]`** - `t.date` - Education graduation date
14. **`@therapist.date_of_birth`** - `t.date` - Birth date
15. **`@preference.projected_start_date`** - `t.date` - Projected start date
16. **`@professional_history.malpractice_liability_insurance_expiration_date`** - `t.date` - Insurance expiration

## DateTime Fields (`t.datetime`) - Use `timezone_aware_formated_date`

These fields store timestamp values that can have timezone conversion issues:

1. **`@therapist.created_at`** - `t.datetime` - Therapist signup timestamp - ✅ **Fixed**
2. **`DateTime.current`** - `DateTime` - Current timestamp for form access tracking - ⚠️ **Needs Review**
3. **`therapist.latest_attestation_completed_at`** - `t.datetime` - Attestation completion timestamp - ⚠️ **Needs Review**

