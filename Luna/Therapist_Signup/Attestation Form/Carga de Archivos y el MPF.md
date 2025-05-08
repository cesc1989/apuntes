# Carga de Archivos y el MPF

Todo lo que se monte en la carpeta del therapist en S3 va directo al MPF. Todo. Sin embargo, llegó la petición de enviar el Packet y el Friendly Packet. ¿Por qué no están llegando?

La respuesta corta es sí están llegando.

La respuesta larga es que sí están llegando pero están sobre escribiendo el 1er Packet y el 1er Friendly Packet. Son más archivos. Estos archivos están sobre escribiendo los originales:

- Packet
- Attestation
- Friendly Packet
- Resume (Employment History)

Eso lo podemos evidenciar en la prueba que hice hoy 7 de Mayo de 2025. Cuando completé el AF de un contacto de pruebas en Producción pude ver que los archivos se cargaron en la carpeta `attestation` tal y como se espera.

![[21.af.files.in.s3.png]]

Sin embargo, esa carpeta no se encuentra en el MPF. No se encuentra porque el MPF carga todos los archivos sin carpetas. Lo que pude comprobar es que se están es sobre escribiendo los archivos que tienen el mismo nombre y que correspondían a la carga del Credentialing Form.

![[22.af.files.in.mpf.png]]

## Alternativa de Solución

A los archivos que se cargan como resultado del AF hay que darles un nombre más único. Para ello podría usar el campo `latest_attestation_completed_at`. Este se actualiza con cada Submit del AF que haga el therapist.