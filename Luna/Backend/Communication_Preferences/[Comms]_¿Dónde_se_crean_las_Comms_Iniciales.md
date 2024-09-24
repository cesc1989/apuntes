# [Comms] ¿Dónde se crean las Comms Iniciales?
Estos son los archivos donde se crean las `notification_settings` o el `patient_communication_preference_token` al crear pacientes.

# Omni Actions Create Patient

File path: `app/grimoire/omni/actions/create_patient.rb`
Class: `Omni::Actions::CreatePatient`

# Register new Patient Mutation

File path: `app/graphql/mutations/register_new_patient.rb`
Class: `Mutations::RegisterNewPatient`

Aquí se crea porque se dispara un callback al crear el `PatientCommunicationPreferenceToken`.

# Luxe Customers Patients

File path: `app/admin/customers/patients.rb`
Class:  `Patient`

Aquí se crea porque se dispara un callback al crear el `PatientCommunicationPreferenceToken`.

# Create or Update Draft Patient

File path: `app/services/create_or_update_draft_patient.rb`
Class: `CreateOrUpdateDraftPatient`

Aquí se crea porque se dispara un callback al crear el `PatientCommunicationPreferenceToken`.

