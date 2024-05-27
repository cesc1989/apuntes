# S3 to GDrive Set up - Marketplace

# Working on Onboarding GDrive Upload

Setup these envs:

    GOOGLE_API_KEY="SOME_KEY"
    GOOGLE_DRIVE_ROOT_ONBOARDING_FOLDER_ID="PARENT_FOLDER_IN_GOOGLE_DRIVE_ID"
    ONBOARDING_MARKETPLACE_TOKEN="SOME_TOKEN"
    GOOGLE_SA_DOCUMENT_AGENT_1="BASE64_ENCODED_SERVICE_ACCOUNT_JSON"

To generate the `GOOGLE_API_KEY` follow [this guide](https://developers.google.com/drive/api/quickstart/python).

To get the value for `GOOGLE_DRIVE_ROOT_ONBOARDING_FOLDER_ID`, create a folder in a Google Drive account, enter the folder and copy the ID in the URL.

The URL looks like this one:

    https://drive.google.com/drive/folders/1VMfs0B8owYFMaa0Fa04C-ytLzVlOt2Xp

The last part is the folder's ID. Copy that one.

## Setting up a Service Account

In order to complete the connection to Google Drive and push files, you'll need a Service Account.

Follow [this guide](https://medium.com/@matheodaly.md/using-google-drive-api-with-python-and-a-service-account-d6ae1f6456c2) to generate one and enabling the Google Drive API.

This [other guide](https://medium.com/@hitesh.thakur/how-to-upload-file-into-google-drive-via-python-using-service-account-a81f3ed54c66) explains how to create a service account and explains how to setup the destination folder using these credentials.

Once you got the generated JSON file, you have to minify it and encode it into a base64 string. The base64 value is the one to set in the `GOOGLE_SA_DOCUMENT_AGENT_1` env var.

Use [this web site](https://www.base64encode.org/) to encode the service account data.

## Run the default scheduler for GDrive upload

The background job that runs the GDrive upload runs in the default queue. Run that queue with this command:

    rq worker -c marketplace.settings_rq_worker_default

# Sobre meter un JSON en una variable de entorno

Tenemos esto en `app/marketplace/config/base.py`

    ACCOUNT_JSON: SimpleNamespace = SimpleNamespace(
      ONBOARDING = [base64.b64decode(os.environ.get(f"GOOGLE_SA_DOCUMENT_AGENT_{i}", "")).decode() for i in range(1, 20)]
    )

[Aquí comentan](https://stackoverflow.com/questions/56277661/is-it-possible-to-store-a-json-file-to-an-env-variable-with-dotenv) que se puede generar un JSON dump del contenido json y luego hacer un parse al cargarla desde la variable de entorno pero no me sirve por el código anterior. En esa configuración **se está es haciendo un decodeo de un base64**.

## Prueba y Error

Si trato de encodear usando [base64](https://docs.python.org/3/library/base64.html) pasa esto al pasar el JSON o una string con el JSON:

    >>> s={}
    >>> encoded = base64.b64encode(s)
    >>> TypeError: a bytes-like object is required, not 'dict'
    
    >>> s='{}'
    >>> encoded = base64.b64encode(s)
    >>> TypeError: a bytes-like object is required, not 'str'
## Avance

Usé [esta web](https://www.base64encode.org/) para encodear a base64 el JSON del service account y pude configurar así la variable de entorno `GOOGLE_SA_DOCUMENT_AGENT_1`.

## Encodeando con base64 de Python

Así:
```python
import base64
import json
   
myd = '{}'
result = base64.urlsafe_b64encode(json.dumps(myd).encode())
```

Y para decodear:

    base64.b64decode(result).decode()

pero manda el JSON con los doble backslash. Hay que parsearlo.

