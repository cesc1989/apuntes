# Notas de Desarrollo de KX Modifier

Todo lo que voy descubriendo de Edge a medida que hago esta vaina.

# ¿Cuándo un Episode es Medicare?

En Episode veo esto:

```ruby
class Episode < ApplicationRecord
  # Types of care plan referrals
  scope :medicare,
        -> { joins(:insurance).where(insurances: { key: %w[medicare medicare_advantage] }) } # insurance is Medicare


  # (...)

  # "Straight Medicare" i.e. coverage directly provided by the federal government
  # Centers for Medicare & Medicaid Services (CMS).
  # Not to be confused with commercial payer plans i.e. Medicare Advantage
  def medicare?
    is_medicare_plan_category = payer_plan&.category&.medicare? == true

    # XXX: Remove after new plan categories are live
    is_medicare_insurance = insurance&.medicare? == true
    is_medicare_payer = payer&.name&.starts_with?("Medicare") == true && payer_plan&.category&.government? == true

    is_medicare_plan_category || is_medicare_insurance || is_medicare_payer
  end
end
```

> [!Note]
> **Contexto**
> Alexis le preguntó a Christie: "is this strictly for medicare insurances or does it include medicare advantage?"
> 
> Ella contestó: "Medicare only"

Creo que tocaría hacer un filtro como este:
```ruby
medicare_care_plans = patient.episodes.select(&:medicare?)
```

O mejor un scope nuevo que quite la opción `medicare_advantage`:
```ruby
scope :straight_medicare, -> { joins(:insurance).where(insurances: { key: "medicare" }) }
```

¿Tiene sentido algo así? Creo que sí. Cuando reviso en alpha por posibles Insurances que sean `medicare_x` encuentro tres:
```ruby
Insurance.where("key LIKE ?", "%medicare%").pluck(:id, :key)

[[3, "medicare"], [28, "medicare_advantage"], [396, "medicare_secondary"]]
```

## Insurance que es Medicare

En el modelo Insurance veo estos métodos:
```ruby
class Insurance < ApplicationRecord
  def medicare?
    key == "medicare"
  end

  def medicaid?
    key == "medicaid"
  end

  def medicare_advantage?
    key == "medicare_advantage"
  end
end
```

# Estados de Appointment

## Estado Activo

Los appointments están activos cuando están en alguno de estos estados:
```ruby
ACTIVE_STATES = %w[pending ongoing completed].freeze
```

# Usando GraphiQL

Se accede desde `http://localhost:3000/graphiql`. Hay que primero iniciar sesión como admin en Luxe.

## Entidades para pruebas en Local

Necesito:

- Un `Patient` con `episodes` que sean Medicare
- Un `Therapist` ligado a los `appointments` de los `episodes`
- Al menos un `Episode` con `MedicareDollarThresholdStatus`

Episode:
```ruby
Episode.straight_medicare.first.id

Episode.find("0175f881-fd3a-48d5-bbe1-0ef4c259d777")
```

appointments del Episode:
```ruby
Episode.find("0175f881-fd3a-48d5-bbe1-0ef4c259d777").appointments
```

Actualiza el `scheduled_date` del `appointment` para generar un MedicareDollarThresholdStatus:
```ruby
e1.appointments.first.update(scheduled_date:DateTime.current + 7.days)
```

Therapist:
```ruby
Episode.find("0175f881-fd3a-48d5-bbe1-0ef4c259d777").appointments.first.therapist

# => "22520602-2822-4067-a56a-3d163b46ff03"

Therapist.find("22520602-2822-4067-a56a-3d163b46ff03")
```

Credenciales para hacer peticiones:
```ruby
Therapist.find("22520602-2822-4067-a56a-3d163b46ff03").account.create_new_auth_token
```

MedicareDollarThresholdStatus:
```ruby
Episode.find("0175f881-fd3a-48d5-bbe1-0ef4c259d777").medicare_dollar_threshold_status
```

## Ejecutar Queries en GraphiQL

Di con esta forma de hacer una query para ver los datos de un paciente según lo que se define en el tipo.

### Query de patient (vieja forma)

```ruby
# app/graphql/types/query.rb
class Types::Query < Types::BaseObject
  field :patient, Types::Patient, null: false,
                                  authorize: true,
                                  deprecation_reason: "Redundant with `node`" do
    description "Look up a patient by ID."

    argument :id, ID, required: true do
      description "The ID of the patient."
    end
  end
end

# app/graphql/types/query.rb
class Types::Patient < Types::BaseObject
	field :first_name, GraphQL::Types::String, null: false do
    description "First name of this user."
  end

  field :care_plans, [Types::CarePlan], null: false do
  end
end
```

La query:
```json
{
  patient(id: "03da7897-7be3-4102-ac66-956ae61ac73d") {
    id,
    firstName,
    carePlans {
      id
    }
  }
}
```

La respuesta:
```json
{
  "data": {
    "patient": {
      "id": "03da7897-7be3-4102-ac66-956ae61ac73d",
      "firstName": "nippy wisteria akbash",
      "carePlans": [
        {
          "id": "ee51dc7f-ca06-4833-aaa6-4abf614fc59d"
        }
      ]
    }
  }
}
```

### Query usado Node

Tienen en Edge esta forma de hacer las queries a varios modelos:
```json
// el id es del CarePlan/Episode
{
  node(id: "ee51dc7f-ca06-4833-aaa6-4abf614fc59d") {
    ... on CarePlan {
      id,
      createdAt,
      name,
      payerPlan {
        id
      }
    }
  }
}
```

Responde:
```json
{
  "data": {
    "node": {
      "id": "ee51dc7f-ca06-4833-aaa6-4abf614fc59d",
      "createdAt": "2025-02-06T06:21:47-08:00",
      "name": "Ankle/Foot (04/05/2025 - 04/05/2025)",
      "payerPlan": {
        "id": "ea64e469-dbf1-4cd0-9ce0-4d2e17db978e"
      }
    }
  }
}
```


## Probando el campo `promptTherapistForMedicalNecessity`

Cuando pruebo la query de esta forma:
```json
{
  node(id: "ee51dc7f-ca06-4833-aaa6-4abf614fc59d") {
    ... on CarePlan {
      id,
      createdAt,
      name,
      payerPlan {
        id
      },
      promptTherapistForMedicalNecessity
    }
  }
}
```

Da este error:
```json
{
  "errors": [
    {
      "message": "undefined method `values' for #<BatchLoader::GraphQL:0x0000000118527100 @batch_loader=#<BatchLoader:0x1317560>>\n\n        context_key = context.values.map { _1._policy_cache_key(use_object_id: true) }.join(\".\")\n                             ^^^^^^^",
      "backtrace": [
        "/Users/francisco/.gem/ruby/3.1.6/gems/action_policy-0.6.0/lib/.rbnext/1995.next/action_policy/behaviours/policy_for.rb:62:in `policy_for_cache_key'",
        "/Users/francisco/.gem/ruby/3.1.6/gems/action_policy-0.6.0/lib/action_policy/behaviours/memoized.rb:37:in `__policy_memoize__'",
        ]
    }
  ]
}       
```

¿Qué lo causa?

## Impersonate / Suplantar

En ocasiones, será mejor suplantar a algún usuario de tipo Therapist para poder hacer las peticiones de manera correcta.

Para hacerlo hay que usar el correo del Therapist. En mí caso, para probar estas queries, suplanté el therapist con correo `demo+staging26@koombea.com`.

## Ejecutar Mutaciones en GraphiQL

TBC.

# Peticiones en GraphQL desde Postman

El endpoint para probar es `POST /graphql`. Hay que pasar las siguientes cabeceras de autenticación y authorización:
```bash
access-token:eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJmZmZlMzMyYy01Yzk1LTQwZGYtYjg5NC03NTgxM2ZmYTcwOTciLCJpYXQiOjE3NDEyNzA4NzR9.a-yBDmfbT2nBtcDRxwCtCKrDTRFVGV_K3ZJsWgLNrlQ

token-type:Bearer

client:Fdr3O1qtG-PP_oe0-mcNYQ

expiry:1801270874

uid:francisco.quintero+26@ideaware.co

Authorization:Bearer eyJhY2Nlc3MtdG9rZW4iOiJleUpoYkdjaU9pSklVekkxTmlKOS5leUp6ZFdJaU9pSm1abVpsTXpNeVl5MDFZemsxTFRRd1pHWXRZamc1TkMwM05UZ3hNMlptWVRjd09UY2lMQ0pwWVhRaU9qRTNOREV5TnpBNE56UjkuYS15QkRtZmJUMm5CdGNEUnh3Q3RDS3JEVFJGVkdWX0szWkpzV2dMTnJsUSIsInRva2VuLXR5cGUiOiJCZWFyZXIiLCJjbGllbnQiOiJGZHIzTzFxdEctUFBfb2UwLW1jTllRIiwiZXhwaXJ5IjoiMTgwMTI3MDg3NCIsInVpZCI6ImZyYW5jaXNjby5xdWludGVybysyNkBpZGVhd2FyZS5jbyJ9
```

> [!Note]
> Que no haya espacios entre la llave y el valor.

Y se elige la opción "GraphQL" en la pestaña "Body" de la petición. Ahí se pega la query como en GraphiQL.

## Generando las cabeceras

Hay que encontrar un Therapist que tenga acceso a un Care Plan. Y luego se genera la cabecera así:
```ruby
Therapist.find("2f3a6790-a624-44ab-818e-aa961bb1eb82").account.create_new_auth_token
```

Eso devuelve un hash con las cabeceras y sus valores.

## GraphQL esquema Relay y Fragmento *inline*

Una cosa que me complicó probar las queries en local es que no sabía cómo hacer una query para sacar datos de un Care Plan. Al final la query tenía que ser así:
```json
{
  node(id: "ee51dc7f-ca06-4833-aaa6-4abf614fc59d") {
    ... on CarePlan {
      id
    }
  }
}
```

Si bien el ID es de un Care Plan, no sabía como hacer para pasarle los atributos que quería de vuelta. Me di cuenta al revisar una query en unas pruebas y noté el `... on CarePlan {}`.

Le pregunté a ChatGPT sobre eso y me dice que eso es un fragmento condicional. Si el nodo devuelto es de tipo "CarePlan", trae los atributos que indica la query.

En los [docs de GraphQL](https://graphql.org/learn/queries/#inline-fragments) le llaman "inline fragments".

# KX y Candid

## Ingreso y Marcado de `threshold_exceeded`

La integración con Candid funciona leyendo un archivo CSV que ponen en un bucket de S3. Ahí hay varios valores que se importan en la base de datos de Edge. En este parte del import es donde se va a buscar la posible razón de denegación del servicio cuando el paciente ya excedió el límite de gasto del año para Medicare.

Dicho valor se va a encontrar en la columna/header `denial_reasons`.

La documentación de Candid [menciona](https://docs.joincandidhealth.com/api-reference/service-lines/v-2/create) que los valores que puede tener esa columna son:

- Authorization Required
- Referral Required
- Medical Records Requested
- Timely Filing
- Duplicate Claim
- Incorrect Place of Service
- Incorrect Patient Gender
- Bundled
- Exceeded Billable Time
- Invalid Provider Information
- Invalid Diagnosis Code
- Incorrect Procedure Code
- Invalid Modifier
- Missing NDC Code
- Invalid Insurance Data
- No Active Coverage
- Coordination of Benefits
- Incorrect Payer
- Credentialing
- No Effective Contract
- Missing W-9
- Missing Contract Linkage
- Non-Covered Benefit
- Experimental Procedure
- Not Medically Necessary
- Info Requested from Provider
- Info Requested from Patient
- Billing Error
- Unknown
- **Max Benefit Reached**

La que nos interesa es precisamente la última: "Max Benefit Reached". Cuando `denial_reason` es "Max Benefit Reached" hay que buscar el registro `MedicareDollarThresholdStatus` (MDTS) que tenga dicho `Appointment`. A ese MDTS se le cambia el valor del `threshold_exceeded` a `true`.

---

Según mis apuntes, todo este viaje empieza en el worker `Invoicing::Preparation::ImportInvoiceItemsFromCandidWorker`. Este lo que hace es invocar la clase `Candid::ImportInvoiceItemsFromCandidService`. En esta clase, en el método `sync` pasan muchas cosas.

### Explorando `Candid::ImportInvoiceItemsFromCandidService#group_rows_by_appointment_with_annotations`

Este método lo que hace es tomar un hash que tiene los valores del CSV de Candid agrupados por `Appointment` de Edge. El resultado final es algo como esto:
```ruby
(byebug) ap rows_data_by_appointment
{
  "9e7c9651-c8fb-4899-9c39-e782ed9c6a7e" => {
    :has_ever_been_invoiceable? => true,
    :latest_export_csv_row => {
      "encounter_id" => "0912_5678_candid_encounter_id",
      "encounter_external_id" => "9e7c9651-c8fb-4899-9c39-e782ed9c6a7e",
      "patient_external_id" => "d1aca30a-2baf-4f34-a57e-6c5918cd9690",
      "claim_id" => "1234_5678_candid_claim_id",
      "status" => "finalized_paid",
      "created_datetime" => "2024-01-11 03:05:18.296578",
      "state_transition_datetime" => "2024-01-13 11:34:50.060107",
      "do_not_bill" => "false",
      "payer_name" => "SOME PAYER NAME CANDID CAME UP WITH NOT IN OUR SYSTEM WE QUERY FOR THIS",
      "payer_id" => "33f68c66-2b3a-453b-b633-206e5dcb2f03",
      "insurance_type" => "",
      "date_of_service" => "2024-01-09",
      "appointment_type" => "",
      "place_of_service_code" => "12",
      "charge_amount_cents" => "28000",
      "allowed_amount_cents" => "9884",
      "paid_amount_cents" => "0",
      "patient_responsibility_cents" => "9884",
      "copay_cents" => "0",
      "coinsurance_cents" => "0",
      "deductible_cents" => "9884",
      "patient_payments_cents" => "3000",
      "patient_balance_cents" => "6884",
      "diagnosis_codes" => "some ICD10 diagnosis codes separated by semicolons",
      "procedure_codes" => "some CPT codes separated by semicolons",
      "claim_adjustment_reason_codes" => "some claim adjustment reason codes separated by semicolons",
      "remittance_advice_remark_codes" => "",
      "denial_reasons" => "None",
      "latest_check_date" => "",
      "latest_check_number" => "",
      "billable_status" => "BILLABLE",
      "responsible_party" => "INSURANCE_PAY",
      "billing_provider_first_name" => "",
      "billing_provider_last_name" => "",
      "billing_provider_organization_name" => "Luna Care, Inc. — Billing Name",
      "billing_provider_tax_id" => "XX-XXXXXXX",
      "billing_provider_npi" => "9575893155",
      "rendering_provider_first_name" => "first2_name902",
      "rendering_provider_last_name" => "last2_name254",
      "rendering_provider_organization_name" => "",
      "rendering_provider_tax_id" => "",
      "rendering_provider_npi" => "3143776513",
      "service_facility_organization_name" => "Luna Care, Inc. (Bay Area)",
      "service_facility_city" => "Lanellton",
      "service_facility_state" => "CA",
      "service_facility_zip_code" => "94102",
      "adjudicated_network_status" => "in_network",
      "organization_id" => "1334c399-15fe-42df-93c7-b3966123e6a1"
    }
  }
}
```

Donde la llave del Hash es el ID del `Appointment` y el valor es otro hash donde se incluyen los valores que vienen desde Candid.

Es en el subhash `latest_export_csv_row` de donde se puede extraer el valor para `denial_reasons`.

# Sendbird

Cuando la Medical Necessity es rechazada hay que enviar un mensaje a Clinical desde Sendbird.

> If confirmed, ==Message will be shared **with Clinical** in Sendbird==. “FirstName LastName has indicated that care is no longer medically necessary for the patient, FirstName LastName.”

Encontré una clase que interactúa con Sendbird en `app/services/sendbird.rb`. Aquí hay varios métodos para enviar mensajes como:

- `send_system_message`
- `send_ticket_message`
- `create_clinical_conversation`

Le pedí ayuda a Cursor y generó algo que luego modifiqué para ajustarlo más a la realidad:
```ruby
if medical_necessity_state_kind.rejected?
	patient = appointment.patient
	therapist = current_user.therapist
	conversation = Conversation.find_by(patient: patient, therapist: therapist)

	if conversation&.sendbird_id.present?
		message = "#{therapist.name} has indicated that care is no longer medically necessary for the patient, #{patient.name}."
		Sendbird.send_system_message(conversation.sendbird_id, message)
	end
end
```

Me queda la duda en dos cosas:

- ¿Necesito buscar un `Conversation`?
- ¿Hay que usar `send_system_message`?

## ¿Necesito buscar un `Conversation`?

Sí. Al parecer, este es el modelo donde se guarda la interacción entre usuarios en Sendbird.

En el worker `PainLevelNotifierWorker` se puede ver:
```ruby
class PainLevelNotifierWorker < ApplicationWorker
  # Send an alert when pain level is 8 or higher
  def perform(workout_id)
    workout = Workout.find(workout_id)
    therapist_id = workout.exercise_program.assigned_by
    patient = workout.exercise_program.episode.patient
    conversation = Conversation.find_by(patient_id: patient.id, therapist_id: therapist_id)

    if conversation.present?
      Sendbird.send_system_message(
        conversation.sendbird_id,
        "#{patient.name} just performed a workout and had a pain rating of #{workout.pain_level}/10."
      )
    end
  end
end
```

El modelo Conversation tiene un método para empezar una conversación en Sendbird:
```ruby
def create_sendbird_conversation
	SendbirdConversationCreationWorker.perform_async(self.id)
end
```

## ¿Usar `send_system_message`?

No. Dijeron que eso no es lo correcto porque eso lo que hace es enviar un mensaje al paciente:

![[300.sendbird.message.to.patient.png]]

> The requirements are not to send a message to the patient. It should be a message to Concierge Team: Utilization Management.

¿Cómo se hace eso?

## Enviar mensaje a Clinical Team: Utilization Management



