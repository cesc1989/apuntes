# Notas de Desarrolo del Extended Sign Up

## Asociar y reasociar

Esto tiene que hacerse en un solo paso porque un Contacto solo puede tener un Credentialing Active y un solo Active Attested.

Error que dio intentar asociar sin haber reasociado:
```ruby
{
    "status" => "COMPLETE",
    "results" => [],
    "numErrors" => 1,
    "errors" => [
        [0] {
              "status" => "error",
            "category" => "VALIDATION_ERROR",
             "message" => "Exceeded association limit of 1 for portalId 7712148, fromObjectId 113207996846, associationCategory USER_DEFINED, associationTypeId 53"
        }
    ],
      "startedAt" => "2025-05-14T00:36:12.483Z",
    "completedAt" => "2025-05-14T00:36:12.573Z"
}
```

## Orden de reasociación

En condiciones normales, el Contact tendrá un Credentialing Active Attested. Por esto, para poder asociar el nuevo Credentialing que se crea primero hay que reasociar el existente.

Para eso debo usar la clase `ReassociateActiveAttestedCredentialingObjectService`:
```ruby
assoc = HubspotCustomObjects::Associations::ReassociateActiveAttestedCredentialingObjectService.new(therapist)
assoc.from_active_attested_to_active
assoc.remove_active_attested_label
```

Al ejecutar esos dos métodos un Therapist con esta configuración de hubspot_ids:
```ruby
{
         :therapist_address_hubspot_id => 26239535543,
                   :license_hubspot_id => 26238789130,
             :credentialing_hubspot_id => nil,
    :credentialing_inactive_hubspot_id => nil,
     :credentialing_active_attested_id => 27727231013,
          :credentialing_relocation_id => 1
}
```

> [!Tip]
> `credentialing_hubspot_id` es el ID para el Credentialing con label Active.

Pasa a esta configuración:
```ruby
{
         :therapist_address_hubspot_id => 26239535543,
                   :license_hubspot_id => 26238789130,
             :credentialing_hubspot_id => 27727231013,
    :credentialing_inactive_hubspot_id => nil,
     :credentialing_active_attested_id => nil,
          :credentialing_relocation_id => 1
}
```

Con esto lo que logramos es quitar la asociación Active Attested y pasarla a una Active.

Así es que podemos pasar al siguiente paso que es asignar el nuevo Credentialing como Active Attested con la nueva clase `AssociateCredentialingToContactService`.

A esta clase hay que pasarle el Credentialing ID más reciente para poder:
- Indicar el objeto Credentialing a asociar
- Poder actualizar el campo `credentialing_active_attested_id` correctamente

### Asociando nuevo Credentialing como Active Attested

Cuando todo está en su lugar, la configuración final de hubspot_ids queda así:
```ruby
{
         :therapist_address_hubspot_id => 26239535543,
                   :license_hubspot_id => 26238789130,
             :credentialing_hubspot_id => 27727231013,
    :credentialing_inactive_hubspot_id => nil,
     :credentialing_active_attested_id => 27772111464,
          :credentialing_relocation_id => 1
}
```

Nota que:

- `:credentialing_hubspot_id => 27727231013,` tiene el anterior active attested ID
- `:credentialing_active_attested_id => 27772111464,` es el nuevo Credentialing recién creado

