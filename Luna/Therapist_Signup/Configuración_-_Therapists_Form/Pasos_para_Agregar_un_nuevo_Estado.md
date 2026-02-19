# Pasos para Agregar un nuevo Estado

Cuando quiero agregar un nuevo estado donde Luna va a estar disponible hay que montar el archivo PDF del acuerdo de terapeutas a S3 (uno por cada entorno).

## Rake tasks a ejecutar

Hay que ejecutar la siguiente rake task:
```bash
rake service_states:add_new_service_state[nuevo_estado]
```

## Archivos a subir a S3

El texto del agreement, que normalmente lo entregan en documento word, hay que convertirlo en un PDF y luego ese archivo subirlo a los buckets de cada entorno.

> [!Note]
> El nombre del archivo debería seguir la forma “[estado]_agreement.pdf”, ejemplo: “colorado_agreement.pdf”.

Alpha
```
aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements.new.address/massachusetts_agreement.pdf s3://luna-staging-therapist-signup-agreement-pdfs/
```

Omega
```bash
aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements.new.address/massachusetts_agreement.pdf  s3://therapist-cred-agreements-pdfs/
```

## Relacionados

Listado de estados activos [[Therapist_Signup_Enabled_States]]