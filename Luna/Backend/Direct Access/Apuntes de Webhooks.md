# Apuntes de Webhook de Therapist

Rutas:
```ruby
# incoming hubspot webhooks
post "patient/merge_patients"
post "patient/sync_merged_contacts"
post "physician/sync"
post "physician/sync_merged_contacts"
post "therapist/sync"
post "therapist/sync_merged_contacts"
post "draft_therapist/create"
```

## Actualizar Therapist

Controlador: `app/controllers/therapist_controller.rb`

Pruebas: `spec/requests/therapist_controller_spec.rb`

Workflow en Alpha: https://app.hubspot.com/workflows/7712148/platform/flow/40997250/edit

## Crear Therapist Draft

Controlador: `app/controllers/draft_therapist_controller.rb`

Workflow en alpha: https://app.hubspot.com/workflows/7712148/details/38123686/performance

El controlador define esto:
```ruby
# Create a draft therapist if that therapist does not already exist
class DraftTherapistController < IncomingHubspotWebhookController
  before_action { valid_contact_type?("Therapist") }
  before_action { valid_mandatory_property?("firstname") }
  before_action { valid_mandatory_property?("lastname") }
  before_action { valid_mandatory_property?("email") }
  before_action { valid_mandatory_property?("gender") }
  before_action { valid_mandatory_property?("phone") }
  before_action { valid_mandatory_property?("date_of_birth") }
  before_action { valid_mandatory_property?("desired_weekly_appointments") }
  before_action { valid_mandatory_property?("desired_signature") }
  before_action { valid_mandatory_property?("pet_allergies") }
  before_action { valid_mandatory_property?("bank_account_type") }
  before_action { valid_mandatory_property?("specialties_credentialing_form") }

  # Home address
  before_action { valid_mandatory_property?("address") }
  before_action { valid_mandatory_property?("city") }
  before_action { valid_mandatory_property?("state") }
  before_action { valid_mandatory_property?("zip") }

  # Treatment address
  before_action { valid_mandatory_property?("treating_street_address") }
  before_action { valid_mandatory_property?("treating_city") }
  before_action { valid_mandatory_property?("treating_state") }
  before_action { valid_mandatory_property?("treating_postal_code") }

  # (...)
end
```

Lo que significa que el Therapist solo se creará cuando se haya completado todo el Credentialing Application.

### Workflow

El workflow pedía que la propiedad "EMR Created" estuviera definida. No sé razón de qué pero esa propiedad nunca se llenaba para los therapists creados por el Signup Form así que nunca se crearon en Luxe Alpha.

Decidí quitar esa propiedad y ahora se han creado un montón que antes no habría.

