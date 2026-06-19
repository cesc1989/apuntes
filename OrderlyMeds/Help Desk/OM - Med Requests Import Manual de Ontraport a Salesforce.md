# Importar manual Medication Requests de Ontraport a Salesforce

Para el caso [[OM Ciclo 48#Caso OM-9334 - Check In Error - hk connect 🟡ℹ️]]

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

> [!warning]
> Esto se trata de traer datos de una importación. Por lo tanto hay que buscar el ID del Script que corresponda al Member Period de cuando se hizo la migración.
>
> Para encontrar el MP y Script relacionados se los MPs migrados de Ontraport. Estos tiene estado "Migrated from Ontraport".
>
> En este caso había dos pero solo uno tenía un Medical Encounter. Entonces ese es el que había que revisar. Dicho MP tiene el Script ID que nos sirve en el campo "Ontraport Script Id"

Para armar esta línea:
```ruby
member_period = member_periods_by_script_id.fetch(mapping.ontraport_script_id.to_i)
```

Por alguna razón hay algo que devuelve el script id como flotante. Usar esta parte manualmente dará error. Es mejor buscar el ID del script que corresponda.
```ruby
member_period = 802328
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

member_period = 802328
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

### En Salesforce

Solo es cuestión de ir al Member Period correspondiente y ubicar la sección "Medication Requests". Debe haber al menos un registro.

![[om_9334.png]]

### En la base de datos

Cuando el script de Ruby se completa normal así aparece el registro de `Salesforce::MedicationRequest`:
```ruby
#<Salesforce::MedicationRequest:0x00007fc45e8f9cb8
  patientid: "001Pm00001Tn216IAB",
  patient__omid__c: "019e805e-1b66-70a0-9de6-7243ae60089d",
  prescribeddate: "2026-03-04 17:24:53.000000000 +0000",
  medicationid: "0itPm0000000BoGIAU",
  omid__c: "019ee17e-5bed-7f6d-98b5-6a88185f1a3f",
  refillsallowed: 0,
  name: "MR-335534",
  previousprescriptionid: nil,
  memberperiodid__c: "a0nPm00000nNVTeIAO",
  status: "Completed",
  createddate: "2026-06-19 20:07:01.000000000 +0000",
  fillquantityunit__omid__c: nil,
  previousprescription__omid__c: nil,
  type: "Order",
  sfid: "0kmPm000001DvJFIA0",
  id: 565450,
  _hc_lastop: "SYNCED",
  _hc_err: nil,
  ontraportimporterversion__c: 1.0,
  ontraportscriptid__c: nil,
  importedfromontraportat__c: "2026-06-19 20:06:55.000000000 +0000",
  medication__omid__c: "019a7eef-2991-7030-93b5-073970b7e85c">
```

Nota que los campos:
```ruby
_hc_lastop: "SYNCED",
_hc_err: nil,
```

Están en forma que last operation es "SYNCED" y err no tiene nada. Eso es buena señal.