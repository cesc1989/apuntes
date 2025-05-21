# Puntos de entrada para la creación de pacientes

Original en Notion.

Patients enter our system in one of these ways:

1. *[Physician Referral]* Physician sends a referral fax (“inbound fax”)
2. *[Therapist Referral]* Therapist sends a patient referral to us from the Referral screen in the Therapist App → Creates a row in `referrals` table in Luxe + creates a HubSpot contact + notifies Concierge via HubSpot
3. *[Patient App account creation]* Patient goes through the onboarding in the app → Contact is created in HubSpot
4. *[Patient Booking Request - App]* Patient completes app onboarding and creates a Booking Request → Creates a row in `referrals` table in Luxe + notifies Concierge via HubSpot
5. *[Patient Booking Request - Web]* Patient creates Booking Request from website (no app) → Same flow as app-based request (in this case, the HubSpot Contact is created from the Marketing Site app)
6. *[New patient cold call]* Patients cold call us → CallRail automatically upserts a contact in HubSpot after patient is on phone with agent for certain amount of time (like 60+ seconds)
7. *[Concierge cold call]* “Patient Lead” Contact is in HubSpot w/o an associated Luxe account; Concierge cold calls and attempts to convert them. Patient would be created after successfully completing the Serviceability flow (which Omni will replace)

# Contexto

Necesito saber cuáles son estos puntos de entrada para poder crear el `PatientCommunicationPreferenceToken` para cada paciente nuevo que ingresa al sistema.

# Navegando Luxe: ¿dónde se crean los pacientes?

Encontré esta clase que maneja la creación del paciente en Hubspot → `HubspotPatientContactCreationService`

La anterior clase es usada en `Mutations::IncomingFaxAddDraftPatient`. A su vez, dentro de `Mutations::IncomingFaxAddDraftPatient` podemos ver una invocación a `CreateOrUpdateDraftPatient`.

En la mutación hay un bloque de decisión para crear o actualizar el paciente:

    patient = params[:patient_id].nil? ? draft_patient_service.create! : draft_patient_service.update!

En `CreateOrUpdateDraftPatient` podemos ver esto:
```ruby
  def create!
      ApplicationRecord.transaction do
        Patient.draft.new(patient_attributes)
          .tap(&:build_patient_record_set)
          .tap(&:save!)
          .tap do |p|
            p.hubspot_contact.update!(luxe_url: p.luxe_admin_url)
          end
      end
    end
```

¿Es este un lugar para agregar la creación del `PatientCommunicationPreferenceToken`?

## ¿Dónde se usa Patient.new o Patient.draft.new?

`Patient.new` →

- `Omni::Actions::CreatePatient` 
- `Mutations::RegisterNewPatient`
- `admin/customers/patients.rb`

`Patient.draft.new` →

- `CreateOrUpdateDraftPatient`

