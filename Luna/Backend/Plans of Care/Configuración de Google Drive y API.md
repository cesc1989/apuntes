# Configuración de Google Drive y API para Charts y POCs

Relacionado con [[Configuración de Templates de Fax y Charts]]

Necesito configurar varias cosas en mí local para poder usar las clases de generación de POCs.

## Configuración de las ENVs

Necesito estas ENVs:
```
GOOGLE_ACCOUNT_TYPE
GOOGLE_CLIENT_EMAIL
GOOGLE_CLIENT_ID
GOOGLE_PRIVATE_KEY
```

Estos valores los puedo obtener cuando se crea la service account en la consola de Google Cloud.

> [!Important]
> Para este caso uso la misma SA que generé para el MPF en Marketplace. De ese tengo el archivo JSON con los valores.

Así quedaron configuradas:
```bash
GOOGLE_CLIENT_ID="XXXXXXXXXXXXXXXXXXXXX"
GOOGLE_CLIENT_EMAIL="something@nalu2023.iam.gserviceaccount.com"
GOOGLE_ACCOUNT_TYPE="service_account"
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nUNCLAVEPRIVADA=\n-----END PRIVATE KEY-----\n"
```

## Configuración de Google Drive

En Google Drive hice varias cosas.

- Creé una nueva carpeta: `luna-faxes-templates`
- Dentro de esta creé la carpeta `temporal`
	- Esta es la que da el id a `temp_storage_folder_id`
- Copié los archivos de los templates de fax y charts desde la carpeta de producción
	- Abrí cada uno y guardé los IDs de la URL para poder lograr la configuración en [[Configuración de Templates de Fax y Charts#Actualizar Setting key `google_template_ids`]]

