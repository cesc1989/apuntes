# ✅ File Conversion to Supported Formats

Notes about this process, how it’s done in Luxe, and code for visual helps.

# Sync Signed Terms PDF to Luxe

This is the code of the lambda in Alpha
```python
import json
import urllib.parse
import boto3
import urllib.request
import os

# (...)

# Gather Env Vars
BACKEND_URL = os.environ["backend_url"]
s3 = boto3.client("s3")

def main(event):
		bucket = event\["Records"\][0]\["s3"\]["bucket"]["name"]
		key = urllib.parse.unquote_plus(event\["Records"\][0]\["s3"\]["object"]["key"], encoding="utf-8")

		if ".pdf" not in key:
				log("This lambda is only supported for PDF files")
				return

		try:
				params = {"patient_id": key.split("/")[0]}

				log(f"Sending request to backend with params: {params}")
				request = urllib.request.Request(
						BACKEND_URL,
						data=bytes(json.dumps(params), encoding="utf-8"),
						headers={"Content-type": "application/json", "Accept": "application/json"},
				)
				
		 # (...)
```

The value for the env var:

    BACKEND_URL = os.environ["backend_url"]
    backend_url = https://api2.alpha.getluna.com/api/v2/external/forms/signed_terms

Para peticiones POST esto es clave:

    request = urllib.request.Request(
                BACKEND_URL,
                data=bytes(json.dumps(params), encoding="utf-8"),
                headers={"Content-type": "application/json", "Accept": "application/json"},
            )

Hay que envolver los parámetros en una llamada a `bytes` por [esto](https://docs.python.org/3/library/urllib.request.html#urllib.request.Request):

> For an HTTP POST request method, data should be a buffer in the standard application/x-www-form-urlencoded format. The [urllib.parse.urlencode()](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urlencode) function takes a mapping or sequence of 2-tuples and returns an ASCII string in this format. It should be encoded to bytes before being used as the data parameter.


# File Conversion URLs

    FILE_CONVERSION_URL - ALPHA
    https://therapist-signup.alpha.getluna.com/v2/external/file_conversion
    
    FILE_CONVERSION_URL - OMEGA
    https://success-api.getluna.com/v2/external/file_conversion

