# Aclaración sobre los campos y propiedades de License

Los campos `license_expiration_date` y `license_number` existen en dos modelos diferentes:

```ruby
class Therapist < ApplicationRecord
  validates :physical_therapy_license_number, presence: true
  validates :physical_therapy_license_expiration_date, presence: true
end

class CredentialingInformation < ApplicationRecord

# (...)

# == Schema Information
#  physical_therapy_license_expiration_date        :date
#  physical_therapy_license_number                 :string
end
```

Sin embargo, ==los campos en el modelo Credentialing no están en uso desde hace ya varios años==. Los dejé ahí por razones tontas y hoy vinieron a confundirme.

Para el webhook estos campos han sido fuente de confusión y este documento trata de aclarar todo.

## Actualizando HubSpot

Cuando se registra un nuevo Therapist estos campos llenan las propiedades `treating_license_number` y `treating_license_expiration_date`.

Esto lo podemos ver en las clases:

- `HubspotCustomObjects::Payloads::HubspotSignupPayload` para el caso del Credentialing object.
- y en `HubspotCustomObjects::Payloads::HubspotCredentialingPayload` para el Credentialing object también.

Estos mismos campos también actualizan las propiedades `license_number` y `license_expiration_date` del License object cuando tiene label Treating.