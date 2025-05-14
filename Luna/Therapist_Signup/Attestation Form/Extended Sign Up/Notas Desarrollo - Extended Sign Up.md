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

