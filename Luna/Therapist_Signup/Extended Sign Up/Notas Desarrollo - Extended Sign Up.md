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

Cuando la asociación nueva se completa esta es la respuesta de HubSpot:
```ruby
{
         "status" => "COMPLETE",
        "results" => [
        [0] {
            "fromObjectTypeId" => "2-33642689",
                "fromObjectId" => 27772111464,
              "toObjectTypeId" => "0-1",
                  "toObjectId" => 113207996846,
                      "labels" => [
                [0] "Active Attested"
            ]
        }
    ],
      "startedAt" => "2025-05-14T04:18:36.284Z",
    "completedAt" => "2025-05-14T04:18:36.372Z"
}
```

## Guardado de datos del Sign Up form y mostrarlo como exitoso

Ahora mismo con todo en su lugar, el Sign Up devolverá el error del email tomado y no generará archivos ni redirecciona al Credentialing Form. Hay que ajustar todo eso para que:

- Cuando se dé el Extended Sign Up Check como `true`
	- No devuelva el error de email tomado
	- Genere los archivos al completar `ExtendedSignUpWorker`
	- Redirrecione al Credentialing Form/Attestation Form

### Contexto

Hice una prueba y el Extended Sign Up completó todo lo de HubSpot pero el form devolvió todo esto.

Respuesta en Postman 422 con cuerpo:
```json
{
  "errors": {
    "email": [
      "has already been taken"
    ]
  }
}
```

El endpoint terminó con error y se detuvo:
```bash
  TRANSACTION (0.5ms)  BEGIN
  ↳ app/controllers/api/v1/therapists_controller.rb:12:in `create'
  Therapist Exists? (5.7ms)  SELECT 1 AS one FROM "therapists" WHERE "therapists"."email" = $1 LIMIT $2  [["email", "devtoalpha19@gmail.com"], ["LIMIT", 1]]
  ↳ app/controllers/api/v1/therapists_controller.rb:12:in `create'
  ServiceState Load (1.2ms)  SELECT "service_states".* FROM "service_states" WHERE "service_states"."enabled" = $1 AND "service_states"."name" = $2 LIMIT $3  [["enabled", true], ["name", "new-jersey"], ["LIMIT", 1]]
  ↳ app/models/therapist.rb:72:in `registered_from?'
  TRANSACTION (0.5ms)  ROLLBACK
  ↳ app/controllers/api/v1/therapists_controller.rb:12:in `create'
  Therapist Load (1.0ms)  SELECT "therapists".* FROM "therapists" WHERE "therapists"."email" = $1 LIMIT $2  [["email", "devtoalpha19@gmail.com"], ["LIMIT", 1]]
  ↳ app/lib/extended_sign_up/extended_sign_up_check.rb:5:in `initialize'

2025-05-14T04:41:48.430Z pid=51424 tid=yuk INFO: Sidekiq 7.0.2 connecting to Redis with options {:url=>"redis://127.0.0.1:6379", :size=>5, :pool_name=>"internal"}
2025-05-14T04:41:48.435Z pid=51425 tid=red class=ExtendedSignUp::ExtendedSignUpWorker jid=07d3ae61cb222219d90e3fa3 INFO: start

Failed to sign up new therapist
{:errors=>{:email=>["has already been taken"]}, :postal_code=>"91723", :registered_from=>"new-york", :therapist_name=>"PruebaDate01 PruebaDate01", :tags=>{:registered_from=>"new-york"}}
method=POST path=/v1/therapists format=json controller=Api::V1::TherapistsController action=create status=422 allocations=75483 duration=740.98 view=0.58 db=37.72
```

Pero el ExtendedSignUpWorker hizo lo suyo:
```bash
Therapist Load (1.5ms)  SELECT "therapists".* FROM "therapists" WHERE "therapists"."id" = $1 LIMIT $2  [["id", "e33174e4-7263-4aa6-812e-a9ce85b7796e"], ["LIMIT", 1]]
  ↳ app/workers/extended_sign_up/extended_sign_up_worker.rb:6:in `perform
  Setting Load (0.4ms)  SELECT "settings".* FROM "settings" WHERE "settings"."key" = $1 LIMIT $2  [["key", "credentialings_objectTypeId"], ["LIMIT", 1]]
  ↳ app/services/hubspot_custom_objects/hubspot_credentialing_object_service.rb:94:in `create_params
  Setting Load (0.6ms)  SELECT "settings".* FROM "settings" WHERE "settings"."key" = $1 LIMIT $2  [["key", "credentialings_name"], ["LIMIT", 1]]
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:13:in `initialize
  Setting Load (0.3ms)  SELECT "settings".* FROM "settings" WHERE "settings"."key" = $1 LIMIT $2  [["key", "credentialings_to_contact_active_type_id"], ["LIMIT", 1]]
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:29:in from_active_attested_to_active
  
  TRANSACTION (0.7ms)  BEGIN
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:48:in from_active_attested_to_active
  Therapist Update (4.3ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_hubspot_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-05-14 04:41:49.989183"], ["credentialing_hubspot_id", 27772111464], ["id", "e33174e4-7263-4aa6-812e-a9ce85b7796e"]]
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:48:in from_active_attested_to_active
  TRANSACTION (1.0ms)  COMMIT
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:48:in from_active_attested_to_active
  Setting Load (0.3ms)  SELECT "settings".* FROM "settings" WHERE "settings"."key" = $1 LIMIT $2  [["key", "credentialings_to_contact_active_attested_type_id"], ["LIMIT", 1]]
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:74:in `remove_active_attested_label
  
  TRANSACTION (0.6ms)  BEGIN
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:93:in remove_active_attested_label
  Therapist Update (0.6ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_active_attested_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-05-14 04:41:50.460242"], ["credentialing_active_attested_id", nil], ["id", "e33174e4-7263-4aa6-812e-a9ce85b7796e"]]
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:93:in remove_active_attested_label
  
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/services/hubspot_custom_objects/associations/reassociate_active_attested_credentialing_object_service.rb:93:in `remove_active_attested_label
  Setting Load (0.4ms)  SELECT "settings".* FROM "settings" WHERE "settings"."key" = $1 LIMIT $2  [["key", "credentialings_name"], ["LIMIT", 1]]
  ↳ app/services/hubspot_custom_objects/associations/associate_credentialing_to_contact_service.rb:17:in `create
  Setting Load (0.2ms)  SELECT "settings".* FROM "settings" WHERE "settings"."key" = $1 LIMIT $2  [["key", "credentialings_to_contact_active_attested_type_id"], ["LIMIT", 1]]
  ↳ app/services/hubspot_custom_objects/associations/associate_credentialing_to_contact_service.rb:51:in `payload
  
  TRANSACTION (0.7ms)  BEGIN
  ↳ app/services/hubspot_custom_objects/associations/associate_credentialing_to_contact_service.rb:29:in `create
  Therapist Update (0.9ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_active_attested_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-05-14 04:41:50.954389"], ["credentialing_active_attested_id", 27779361318], ["id", "e33174e4-7263-4aa6-812e-a9ce85b7796e"]]
  ↳ app/services/hubspot_custom_objects/associations/associate_credentialing_to_contact_service.rb:29:in `create
  TRANSACTION (0.8ms)  COMMIT
  ↳ 
  app/services/hubspot_custom_objects/associations/associate_credentialing_to_contact_service.rb:29:in `create
  
2025-05-14T04:41:50.963Z pid=51425 tid=red class=ExtendedSignUp::ExtendedSignUpWorker jid=07d3ae61cb222219d90e3fa3 elapsed=2.528 INFO: done
```

### Solución

## HubSpot Status Codes

Probando en Alpha obtuve este código en la petición para asociar Credentialing to Contact
```
207	POST	/crm/v4/associations/credentialings/contact/batch/create	May 14, 2025 5:38 PM EDT
```

¿Qué rayos es 207? Los docs dicen esto:

> `207 Multi-Status`: returned when there are different statuses (e.g., errors and successes)

O sea que algo falló pero también hubo errores.

### Contexto

No hubo ningún error grave. Falló una validación en la clase `ReassociateActiveAttestedCredentialingObjectService`:
```ruby
def from_active_attested_to_active(skip_relocation_check: false)
	# Cannot make request to reassociate labels
	return false if @therapist.credentialing_active_attested_id.blank?

	# The reassociation already happened
	return false if @therapist.credentialing_active_attested_id.present? && @therapist.credentialing_relocation_id.blank?
```

Como en Extended Sign Up, lo más probable, es que no haya objeto de Relocation, el atributo `credentialing_relocation_id` esté nulo y se detendrá la ejecución de código.

### Solución: bandera para saltar la validación

El cambio que hice fue el siguiente:
```ruby
def from_active_attested_to_active(skip_relocation_check: false)
	# Cannot make request to reassociate labels
	return false if @therapist.credentialing_active_attested_id.blank?

	# The reassociation already happened
	return false if !skip_relocation_check &&
									@therapist.credentialing_active_attested_id.present? &&
									@therapist.credentialing_relocation_id.blank?
```

Agregué un parámetro que servirá de bandera para decidir si validar o no.

Anoto esto para que me sirva de ayuda para leer esta condición en el futuro.

**En caso del Extended Sign Up y sin `credentialing_relocation_id`**

Cuando `skip_relocation_check: true` y `credentialing_relocation_id.blank?`
```ruby
!skip_relocation_check -> false && true && true
```

El parámetro en `true` funciona como corto circuito y hace que no haya retorno porque el operador `&&` necesita que todas las condiciones sean verdaderas.

**En caso de Relocation Address y con `credentialing_relocation_id`**

Cuando `skip_relocation_check: false`:
```ruby
!skip_relocation_check -> true && true && true
```

No hay corto circuito durante y todas las verificaciones retornan verdadero.