# Importar manual Medication Requests de Ontraport a Salesforce

Para el caso OM-9334.

## Trigger manual de partes de Salesforce::ImportAccountFromOntraportContactJob

Archivo en `app/sidekiq/salesforce/import_account_from_ontraport_contact_job.rb`.

La solución empieza yendo al job `Salesforce::ImportAccountFromOntraportContactJob`. Ubicar el fragmento:
```ruby
contact_import = OntraportMigration.fetch_contact(ontraport_contact_id)
```

Hay que tener el contacto de Ontraport. Para eso hay que buscar el contacto por correo en Ontraport o buscar el ID en los detalles del perfil en Salesforce.

```ruby
contact_import = OntraportMigration.fetch_contact(515176)
```

Y luego vamos a la clase `Salesforce::OntraportAccountImporter` y ubicamos el código correspondiente a Medication Requests:
```ruby
contact_import.medication_request_mappings.each do |mapping|
	member_period = member_periods_by_script_id.fetch(mapping.ontraport_script_id.to_i)
	medication = fetch_medication(mapping.med_id)

	Salesforce::MedicationRequest.where(account:, member_period:).first_or_initialize.tap do |medication_request|
		medication_request.assign_attributes(
			account:,
			member_period:,
			medication:
		)
		medication_request.assign_attributes(mapping.mapped_attributes)

		if medication_request.changed? || !medication_request.persisted?
			medication_request.imported_from_ontraport_at = Time.current
			medication_request.ontraport_importer_version = IMPORTER_VERSION
			medication_request.save!
		end
	end
end
```

Tenemos que preparar fragmentos de código para poder ejecutar este bloque.

> [!Note]
> Lo que se hace aquí es, manualmente, disparar sync de Ontraport a Salesforce.

## Preparación del script Ruby manual

Hay varias funciones problemáticas así que lo mejor es copiar los valores directos y no usarlas.

### Las variables que hay que armar

Se necesitan tres variables:
- `account`
- `member_period`
- `medication`

##### Para `account`

Usamos el ID del CX en Ontraport:
```ruby
account =
	Salesforce::PersonAccount.where(
		ontraport_contact_id: ontraport_contact_id
	).first
```

##### Para `medication` y `medication_request_mappings` 🔑

> [!Note]
> Esto es lo primero a resolver para que pueda correr el bloque de código.

Para este hay que revisar que el Script en Ontraport tenga un valor en el campo `med_id_prescribed` que corresponde al campo "MedIDPrescribed" en la UI de Ontraport.
![[ontraport.medidprescribed.png]]

Si no lo hay, copia el mismo que tiene "MedIDSent". Muy probable que sea el mismo.

##### Para armar `member_period` copia manualmente el ID del Script

Para armar esta línea:
```ruby
member_period = member_periods_by_script_id.fetch(mapping.ontraport_script_id.to_i)
```

Por alguna razón hay algo que devuelve el script id como flotante. Usar esta parte manualmente dará error. Es mejor buscar el ID del script que corresponda.
```ruby
member_period = 5555555
```

##### Sobre la constante IMPORTER_VERSION

Solo hay que reemplaza `IMPORTER_VERSION` por `"1"`.

## Bloque final

Variables:
```ruby
contact_import = OntraportMigration.fetch_contact(515176)
account =
	Salesforce::PersonAccount.where(
		ontraport_contact_id: 515176
	).first

member_period = 5555555
mapping = contact_import.medication_request_mappings.first
medication = Salesforce::Medication.find(mapping.med_id)
```

Y se corre esto:
```ruby
Salesforce::MedicationRequest.where(account:, member_period:).first_or_initialize.tap do |medication_request|
	medication_request.assign_attributes(
		account:,
		member_period:,
		medication:
	)
	medication_request.assign_attributes(mapping.mapped_attributes)

	if medication_request.changed? || !medication_request.persisted?
		medication_request.imported_from_ontraport_at = Time.current
		medication_request.ontraport_importer_version = "1"
		medication_request.save!
	end
end
```

## Revisión de Resultados

