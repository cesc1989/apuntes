# NEX-3452 - Credentialing Therapist sin registro en Marketplace para hacer sync de MPF

Etiquetas: #luna_help_desk 

## Contexto

Kayleen reportó que intentó usar la página para hacer sync de MPF desde Luxe pero no pasó nada luego de más de 5 minutos.

## Problema

Todo estaba en orden. Existía en la tabla `tc_therapist` y tenía el mismo HubSpot ID. En los logs pude ver que se había encolado el worker. El descubrimiento vino cuando revisé logs en Marketplace. Salía que el Therapist no existía en la tabla `therapist_sign_up`.

## Solución

Crear el registro. Pedí prod-op, revisé que se creó y luego hice el sync desde Luxe. A los minutos se corrieron los jobs y se creó el MPF para el PT.