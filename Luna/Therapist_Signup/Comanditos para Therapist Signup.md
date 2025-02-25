# Comanditos para Therapist Signup

Comandos rakes o para usar en la consola de Rails.

## Probar carga de archivos PDF a la carpeta en S3

Versión del Credentialing Application:
```ruby
ResumePdfUploader.new(t1).upload
PacketPdfUploader.new(t1).upload
FriendlyPacketPdfUploader.new(t1).upload
AttestationPdfUploader.new(t1).upload
```

Versión para el Attestation Form:
```ruby
ResumePdfUploader.new(t1, subfolder: "attestation").upload
PacketPdfUploader.new(t1, subfolder: "attestation").upload
FriendlyPacketPdfUploader.new(t1, subfolder: "attestation").upload
AttestationPdfUploader.new(t1, subfolder: "attestation").upload
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
t1 = Therapist.find("1def494c-b0c2-4a4e-ba9b-b0ae2be9eb7d")
c1 = t1.credentialing_information
references = Array.new(3) do
 PersonalReference.new(
   credentialing_information_id: c1.id
 )
end
references.each { |pr| pr.save(validate: false) }
```

## Quema el HS ID de un contact con custom objects.

Para facilitar las pruebas de Attestation Form. Encuentra un HS Contact que ya tenga las asociaciones, copia su ID y quemalo en algún registro de la BD local.

Finalmente, sincroniza los ids de los custom objects en el registro therapist.
```ruby
te = Therapist.find("f79c147e-d402-4c0c-84d4-3d99f8e9689c")
te.update(hubspot_id: 3551751)
HubspotSyncCustomObjectsIdsToTherapistService.new(te).sync_ids
```

## Prueba manual para asociar Personal Reference

```ruby
ent = PersonalReference.find(16)
ref = ProfessionalReference::HubspotProfessionalReferenceContact.new(ent)
ref.find_or_create
```

## Crear Licenses desde la consola

En lugar de abrir el form.
```ruby
License.create([
  { credentialing_information_id: 227, state_name: "Florida", license_number: "1223343", expiration_date: "2030-12-31" },
  { credentialing_information_id: 227, state_name: "California", license_number: "1223343", expiration_date: "2030-12-31" },
  { credentialing_information_id: 227, state_name: "Ohio", license_number: "1223343", expiration_date: "2030-12-31" }
])
```

## Para probar la sincronización de custom objects desde consola

Línea por cada sección y para cada diferente custom object.

```ruby
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :signup).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :personal_information).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :immunization).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :credentialing).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :employment).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :preferences).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :npi_and_caqh).update
HubspotCustomObjects::HubspotCredentialingObjectService.new(t1, section: :certification).update

HubspotCustomObjects::HubspotLicenseObjectService.new(t1, section: :signup).update
HubspotCustomObjects::HubspotLicenseObjectService.new(t1, section: :credentialing).update

HubspotCustomObjects::HubspotAddressObjectService.new(t1, section: :preferences).update

HubspotCustomObjects::HubspotAttestationFormUrlService.new(t1).update
```

## Rake para crear un Therapist en Local sin usar Postman

```bash
bundle exec rake therapist:create_local_for_tests
```

# Otros Comanditos

El de Patient Self Report -> [[Comanditos para Patient Self Report]]