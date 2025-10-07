# 007 - Modify Board Certification answer in Attestation Form

Etiquetas: #luna_help_desk 

Caso EDG-2760

## Problema

Los campos de educación en la sección "Information for Credentialing" tienen una regla que impide modificación en el Attestation Form si fueron llenados desde el Credentialing Application.

El reporte solicita poder cambiar la respuesta Yes/No de la pregunta _"##### Do you currently hold any board or specialty certifications?"_. Cambiar de "yes" a "no" pero no es posible por la lógica existente.

Para poder hacer el cambio hay que modificar manualmente o quitar la lógica.

Estos son los valores de Board Certification para ese Therapist:
```json
"board_certification": true,
"board_certification_copy": "https://luna-omega-workloads-therapist-credentialing.s3.us-west-2.amazonaws.com/SOMEID/board_certification_copy.pdf",
"board_certification_date": "09/28/2024",
"board_certification_expiration_date": "09/28/2026",
"board_certification_name": "Other",
"board_certification_number": "",
"had_board_certification": true,
"had_board_certification_expiration_date": true,
"had_board_certification_number": false,
"other_board_certification_name": "",
```

La clave aquí es el campo `"had_board_certification": true` al estar en true ya no se puede modificar en el Attestation Form.

## Solución 🟢

Modificar manualmente el campo `had_board_certification` a `false` usando un script en Rails console o mediante una petición con Bruno/Postman.

Para ello tuve que primero agregar los campos `had_x` al controlador del endpoint de credentialing information del Attestation Form. Una vez los campos fueron agregados al whitelist, pude hacer la actualización manualmente.