# Pasos para Agregar un nuevo Estado

Cuando quiero agregar un nuevo estado donde Luna va a estar disponible hay que montar el archivo PDF del acuerdo de terapeutas a S3 (uno por cada entorno).

## Crear nuevo registro de `Credentialing::ServiceState`

> [!Note]
> Esto antes era una rake pero después del AppBlend se borraron las rakes. Ahora es solo correr una query active record.
> 
> La rake era esta: https://github.com/lunacare/therapist-credentialing-backend/blob/upgrade_ruby_to_316/lib/tasks/007_handle_service_states.rake

Se crea así:
```ruby
ServiceState.create(
  name: "Iowa",
  enabled: true,
  legal_name: "Iowa Luna Care Physical Therapy, LLC"
)
```


## Archivos a subir a S3

El texto del agreement, que normalmente lo entregan en documento word, hay que convertirlo en un PDF y luego ese archivo subirlo a los buckets de cada entorno.

> [!Note]
> El nombre del archivo debería seguir la forma “[estado]_agreement.pdf”, ejemplo: “colorado_agreement.pdf”.

Alpha
```bash
aws s3 cp ~/Documents/Documents/luna-dev-files/therapists-signup/agreements-pdf/iowa_agreement.pdf s3://luna-alpha-workloads-therapist-signup-agreement-pdfs/
```

Omega
```bash
aws s3 cp ~/Documents/Documents/luna-dev-files/therapists-signup/agreements-pdf/iowa_agreement.pdf s3://luna-omega-workloads-therapist-signup-agreement-pdfs/
```

## Relacionados

Listado de estados activos [[Therapist_Signup_Enabled_States]]