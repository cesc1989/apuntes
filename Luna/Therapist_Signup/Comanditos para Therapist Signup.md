# Comanditos para Therapist Signup

Etiquetas: #comanditos 

Comandos rakes o para usar en la consola de Rails.

## Probar carga de archivos PDF a la carpeta en S3

Versión del Credentialing Application:
```ruby
Credentialing::ResumePdfUploader.new(t1).upload
Credentialing::PacketPdfUploader.new(t1).upload
Credentialing::FriendlyPacketPdfUploader.new(t1).upload
Credentialing::AttestationPdfUploader.new(t1).upload
```

Versión para el Attestation Form:
```ruby
Credentialing::ResumePdfUploader.new(t1, subfolder: "attestation").upload
Credentialing::PacketPdfUploader.new(t1, subfolder: "attestation").upload
Credentialing::FriendlyPacketPdfUploader.new(t1, subfolder: "attestation").upload
Credentialing::AttestationPdfUploader.new(t1, subfolder: "attestation").upload
```

## Convierte archivo a PDF con Image Magick

```ruby
to_pdf = MiniMagick::Image.open("storage/photo.jpg")
to_pdf.format "pdf"
to_pdf.write("storage/photo_converted.pdf")
```

## Crear References por defecto

Estas son las que se cargan en la tabla de "Professional References" en la sección "Information for Credentialing":
```ruby
t1 = Credentialing::Therapist.find("1def494c-b0c2-4a4e-ba9b-b0ae2be9eb7d")
c1 = t1.credentialing_information
references = Array.new(3) do
 Credentialing::PersonalReference.new(
   credentialing_information_id: c1.id
 )
end
references.each { |pr| pr.save(validate: false) }
```

## Quema el HS ID de un contact con custom objects.

Para facilitar las pruebas de Attestation Form. Encuentra un HS Contact que ya tenga las asociaciones, copia su ID y quemalo en algún registro de la BD local.

Finalmente, sincroniza los ids de los custom objects en el registro therapist.
```ruby
te = Credentialing::Therapist.find("f79c147e-d402-4c0c-84d4-3d99f8e9689c")
te.update(hubspot_id: 3551751)
Credentialing::HubspotSyncCustomObjectsIdsToTherapistService.new(te).sync_ids
```

## Prueba manual para asociar Personal Reference

```ruby
ent = Credentialing::PersonalReference.find(16)
ref = ProfessionalReference::HubspotProfessionalReferenceContact.new(ent)
ref.find_or_create
```

## Crear Licenses desde la consola

En lugar de abrir el form.
```ruby
Credentialing::License.create([
  { credentialing_information_id: 227, state_name: "Florida", license_number: "1223343", expiration_date: "2030-12-31" },
  { credentialing_information_id: 227, state_name: "California", license_number: "1223343", expiration_date: "2030-12-31" },
  { credentialing_information_id: 227, state_name: "Ohio", license_number: "1223343", expiration_date: "2030-12-31" }
])
```

## Para probar la sincronización de custom objects desde consola

Línea por cada sección y para cada diferente custom object.

**Para Credentialing object**:
```ruby
Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :signup).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :personal_information).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :immunization).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :credentialing).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :employment).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :preferences).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :npi_and_caqh).update

Credentialing::HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :certification).update
```

Para License object:
```ruby
Credentialing::HubspotCustomObjects::HubspotLicenseObjectService.new(t1, section: :signup).update
Credentialing::HubspotCustomObjects::HubspotLicenseObjectService.new(t1, section: :credentialing).update
```

Para Therapist Address object:
```ruby
Credentialing::HubspotCustomObjects::HubspotAddressObjectService.new(t1, section: :preferences).update
```

Para guardar la URL del AF en el Credentialing object:
```ruby
Credentialing::HubspotCustomObjects::HubspotAttestationFormUrlService.new(t1).update
```

## Sincronización manual de HS Custom Object IDs

Para sincronizar IDs de Credentialing y Therapist Address objects:
```ruby
Credentialing::HubspotSyncCustomObjectsIdsToTherapistService.new(t1).sync_ids
```

Para sincronizar ID de License object (el que es el principal no los de diferentes estados):
```ruby
Credentialing::HubspotSyncCustomObjectsIdsToTherapistService.new(t1).sync_license_id
```


## Rake para crear un Therapist en Local sin usar Postman

Hace lo siguiente:

- Crea el therapist
- Crea el contacto en Hubspot
- Crea Answers para Medicare Requirement
- Crea Credentialing Information y sus respectivas References (3)
- Espera 30 segundos para darle tiempo a Hubspot
- Sincroniza IDs de Credentialing y Therapist Address Custom object

```bash
bundle exec rake therapist:create_local_for_tests
```

# Crea objetos que se crean en el Sign Up

Son:

- Credentialing Information
	- Tres Personal References
- Medicare Requirement
	- Diez answers

## Crear credentialing_information & personal_references

```ruby
therapist = Credentialing::Therapist.find(ID)

Credentialing::SetupCredentialingInformation.new(therapist).create
```

## Crear medicare_requirement & answers

```ruby
therapist = Credentialing::Therapist.find("ID")
Credentialing::SetupMedicareRequirement.new(therapist).create
```

O versión job:
```ruby
Credentialing::SetupMedicareRequirementWorker.perform_async("034dcade-119c-4683-b394-0d09f35b9015")
```

# Otros Comanditos

El de Patient Self Report -> [[Comanditos para Patient Self Report]]

El de Clinical Dashboad -> [[Comanditos para Clinical Dashboard]]