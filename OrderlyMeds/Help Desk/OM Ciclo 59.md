# OM Ciclo 59

> [!Info]
> Del Jueves 5 hasta el Miércoles 18 de Noviembre.


> [!Note]
> En el Ciclo 58 no hice anotaciones.


## Caso OM-12437 - Stuck at PrescriptionWritten SmartPharma

Etiquetas: #om_stuck_in_prescription_written #om_smart_pharma

Caso típico donde el MP queda en PrescriptionWritten y no avanza. Corrí el script de debuggeo y salió así:
```
--- Pharmacy submission records per order ---
  Order 01a15fba-f42c-715d-ab38-833d785f2d6b -> SmartPharma::MedicationOrder 01a15fba-f54c-7fec-9194-a3789896886d
    status:              pending
    external_nk:         07bf0c8b-f2c7-485d-b893-4e5e1b2999b6
    practitioner present: true
    patient present:      false
    medication_requests:  3
    validated_at:        nil
    sent_at:             nil
  NOTE: unlike Casa::Order, SmartPharma::MedicationOrder has NO
  failure_message column and no "failed" status -- a rejection here
  just leaves it stuck at whatever status it was in (pending /
  allergies_submitted / validated), with the actual error only sent to
  Rollbar (fingerprint "smart_pharma_*"). Check there for the real reason.
  
  Order 01a1a843-33cb-7b4f-aaf4-5241e765495d -> SmartPharma::MedicationOrder 01a1a843-3523-72e2-bb6c-8bae560c359e
    status:              pending
    external_nk:         1492fc37-3cb4-4dec-9021-bd5f2b368fa2
    practitioner present: true
    patient present:      false
    medication_requests:  3
    validated_at:        nil
    sent_at:             nil
  NOTE: unlike Casa::Order, SmartPharma::MedicationOrder has NO
  failure_message column and no "failed" status -- a rejection here
  just leaves it stuck at whatever status it was in (pending /
  allergies_submitted / validated), with the actual error only sent to
  Rollbar (fingerprint "smart_pharma_*"). Check there for the real reason.
```

Revisando con Claudio encontramos que el CX en SmartPharma no existía en la BD. En todo caso después hubo problemas con la conexión a la farmacia.

### Verificaciones

Revisando el paciente en la tabla de SmartPharma:
```ruby
mo = SmartPharma::MedicationOrder.find("01a1a843-3523-72e2-bb6c-8bae560c359e")
acct = mo.account
prov = mo.order.provider

pp SmartPharma::Patient.for_account(acct)
```

salida:
```ruby
nil
```


Procedimos a crear el paciente con la API:
```ruby
data = {
  first_name: prov.first_name,
  last_name: prov.last_name,
  email: prov.email,
  phone_number: prov.sms_number,
  date_of_birth: prov.date_of_birth.to_date.to_fs(:iso8601),
  gender: prov.sex_assigned_at_birth,
  address_line1: prov.address,
  address_line2: prov.address2,
  city: prov.city,
  state: prov.state,
  zip: prov.zip,
  account_id: acct.id,
  practitioner_identifier_nk: mo.practitioner&.external_nk
}

r = SmartPharma::Api::Patient.create(
  client: SmartPharma::Api::Client.new,
  patient_data: SmartPharma::Api::Encoder::Patient.encode(patient_data: data)
)
```

Salida:
```ruby
[{"DOB" => "1987-03-03",
    "MI_name" => nil,
    "access_key" => "d9508d5f444df25d745b70196ba1c0cd280b33797d5b3f6c646455f2e29c3e4b",
    "date_created" => "2026-09-01 17:03:25",
    "email" => "jason77700+happy@gmail.com",
    "first_name" => "Jason",
    "is_archived" => 0,
    "last_name" => "Mitchell",
    "link_id" => 10847624,
    "patient_id" => 11299422,
    "phone_primary" => ,
    "sex" => "M",
    "species" => "human"}]
  [{"message" => "Possible duplicate profile found!", "type" => "warning"}]
```

Sin embargo, en SmartPharma fue creado con otro correo: `jason77700+happy@gmail.com`. En cambio en nuestro sistema el email es `jason77700@gmail.com`.

#### Crear registro en SmartPharma::Patient

Igual teníamos que crear el registro en esa tabla:
```ruby
mo   = SmartPharma::MedicationOrder.find("01a1a843-3523-72e2-bb6c-8bae560c359e")
acct = mo.account
prov = mo.order.provider
prac = mo.practitioner

patient = SmartPharma::Patient.new(
  account: acct,
  external_nk: "11299422",
  first_name: prov.first_name,
  last_name: prov.last_name,
  email: prov.email,
  phone_number: prov.sms_number,
  date_of_birth: prov.date_of_birth.to_date,
  gender: prov.sex_assigned_at_birth,
  address_line1: prov.address,
  address_line2: prov.address2,
  city: prov.city,
  state: prov.state,
  zip: prov.zip,
  practitioner_identifier_nk: prac.external_nk
)
patient.patient_practitioners.build(practitioner: prac)
patient.save!
```

#### Paciente en otro portal

En todo caso el problema es que el recurso externo pertenece a otro portal. ¿Cómo así?

Cuando se intenta ver los datos del paciente en SmartPharma:
```ruby
c = SmartPharma::Api::Client.new
SmartPharma::Api::Patient.show(client: c, patient_id: "11299422")
```

da esto:
```ruby
lib/smart_pharma/api/response.rb:15:in 'SmartPharma::Api::Response#initialize': Invalid response: No access to that resource. (SmartPharma::Api::InvalidResponseError)
```

Pasa lo mismo si intento correr:
```ruby
SmartPharma::CreatePatientForMedicationOrderJob.new.perform(mo.id)
```

salida:
```ruby
'SmartPharma::Api::Response#initialize': Invalid response: No access to that resource. (SmartPharma::Api::InvalidResponseError)
```