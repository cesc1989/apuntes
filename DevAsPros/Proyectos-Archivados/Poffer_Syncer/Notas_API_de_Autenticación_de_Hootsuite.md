# Notas: API de Autenticación de Hootsuite
Iba a intentar la autenticación por Postman pero hay que hacerlo por OAuth2 y toca iniciar sesión en la cuenta. En esta [sección del FAQ](https://developer.hootsuite.com/docs/rest-api-faq#how-to-authenticate-via-curl) explican cómo hacerlo por cURL.

## Primer Paso

Ir a esta URL:

    https://platform.hootsuite.com/oauth2/auth?response_type=code&client_id=6df333a9-c5a5-4639-aa90-6d091f5edafb&redirect_uri=https://www.getpostman.com/oauth2/callback

Reemplazando `client_id` y `redirect_uri` por los valores correspondientes.

En la consola de desarrolladores, ir a la pestaña "Network" y buscar la opción "Preserve logs".

Inicia sesión o da "Allow" y busca una petición que sea un 302. Ahí busca la URL de la petición.

Luce así:

    https://app.getpostman.com/oauth2/callback?code=Z4UuVwxdk2LL1nD3_hItv_jZW9vB4bkwOlB-Lutb-Mk.XtGos9KIGsfgBuS1ncMYOcioI81MTfkbZMJta6_oMn0&scope=offline&state=

Copia `code=`.

## Segundo Paso

Haz la petición con cURL

    curl -v -H "Content-Type: application/x-www-form-urlencoded" \
    -u 6df333a9-c5a5-4639-aa90-6d091f5edafb:3v-MOpm2DVf- \
    -d code=75A-ZV-2_t188dHZ93_kNgWCUBLnIyJCWazcrzvxMFk.qXdTG5Fb1Kzc37sYpfsZK8EB583TTU_RZv0xjLkp120 \
    -d grant_type=authorization_code \
    -d redirect_uri=https://www.getpostman.com/oauth2/callback \
    -X POST https://platform.hootsuite.com/oauth2/token

En `-u` los valores son: `client_id`: `secret_id`

Si la petición es buena, habrá un JSON respuesta con el *token* para hacer peticiones:

    {
      "access_token":"sJ3zb54U9KummvAwOOaRuflLwJrAnYbm3PdXtId8dT4.y8MAzT7ipwr071S46jUxF5hfm2053d9lx-BsBy9tL5c",
      "expires_in":3599,
      "refresh_token":"ajPyorJmiIbWjXZg_GKjxBDiacRBEaZ4QxtD_fXVOt8.QEm9TAvPVbFFO-Z5N_2x5GWjGx06sO1fXZNXc2R-QSU",
      "scope":"offline",
      "token_type":"bearer"
    }

El token adquirido de la forma anterior **tiene duración de 1 hora**.

## Notas Primera Prueba con Hootsuite

Para programar un mensaje hay que hacer una petición al endpoint: `POST https://platform.hootsuite.com/v1/messages`

> Cabecera de Authorization lleva el token como Bearer.

Los campos: `text`, `socialProfileIds` y `scheduledSendTime` son obligatorios.

> Sino mando `scheduledSendTime`, no se va a ubicar en las casillas preconfiguradas

Los Social Profile IDs se piden en `GET https://platform.hootsuite.com/v1/socialProfiles`.


## Refrescar Token Vencido

Hay que hacer petición a `POST https://platform.hootsuite.com/oauth2/token` con párametros en modo `x-www-form-urlencoded`:

    grant_type: refresh_token
    refresh_token: <EL TOKEN DE LA ANTERIOR PETICION>

Y cabeceras de autenticación básica con CLIENTE:SECRETO

    Authorization: Basic NmRmMzMzYTktYzVhNS00NjM5LWFhOTAtNmQwOTFmNWVkYWZiOjN2LU1PcG0yRFZmLQ==

**Respuesta**

    {
      "access_token": "XBzbHuK1e3-txXsEHl7erBn8RYGlQVrimhSWeTx9YMs.aSirYm0px23q7ymEoUAPILS8ghe6e6mdVQLZ3wHNqgY",
      "expires_in": 3599,
      "refresh_token": "rQMUZzgte7iRmYdsAY8sOH0PXfgj28DwJ3scObX30XA.VgGMBRbjDy3S9a_4Mr4mMW3DvfYm2Sl1SdVUYdBuhrM",
      "scope": "offline",
      "token_type": "bearer"
    }

Petición cURL para refrescar token:

    curl --location --request POST 'https://platform.hootsuite.com/oauth2/token' \
    --header 'Authorization: Basic NmRmMzMzYTktYzVhNS00NjM5LWFhOTAtNmQwOTFmNWVkYWZiOjN2LU1PcG0yRFZmLQ==' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --header 'Cookie: __hsaneauuid=89118c42-eaf4-4d83-b061-4af0d48187ee.Vzqm0fBccL6rISYidkDBla/yz7iRoK9Q1W5CW+x2yXm2w/5lz6czytKrkZoMuTi7gaMEciB67facBbywnvKjAQ==' \
    --data-urlencode 'grant_type=refresh_token' \
    --data-urlencode 'refresh_token=ajPyorJmiIbWjXZg_GKjxBDiacRBEaZ4QxtD_fXVOt8.QEm9TAvPVbFFO-Z5N_2x5GWjGx06sO1fXZNXc2R-QSU'

