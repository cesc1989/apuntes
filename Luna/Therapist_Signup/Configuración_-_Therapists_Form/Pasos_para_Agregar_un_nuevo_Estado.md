# Pasos para Agregar un nuevo Estado

Cuando quiero agregar un nuevo estado donde Luna va a estar disponible hay que montar el archivo PDF del acuerdo de terapeutas a S3 (uno por cada entorno).

## Rake tasks a ejecutar

Hay que ejecutar la siguiente rake task:

    rake service_states:add_new_service_state[nuevo_estado]

## Archivos a subir a S3

El texto del agreement, que normalmente lo entregan en documento word, hay que convertirlo en un PDF y luego ese archivo subirlo a los buckets de cada entorno.

**NOTA**: el nombre del archivo debería seguir la forma “estado_agreement.pdf”, ejemplo: “colorado_agreement.pdf”.


    # Staging
    aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements.new.address/massachusetts_agreement.pdf s3://luna-staging-therapist-signup-agreement-pdfs/
    
    # Production
    aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements.new.address/massachusetts_agreement.pdf  s3://therapist-cred-agreements-pdfs/


