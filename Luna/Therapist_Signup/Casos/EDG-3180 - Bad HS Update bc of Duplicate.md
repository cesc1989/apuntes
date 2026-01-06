# Registro de Therapist Duplicado causa update incorrecto en HubSpot

Etiquetas: #luna_help_desk 

## Contexto

Un Therapist (En Luxe) que tenía duplicado fue mezclado varias veces. Esto dejó al final el contacto en HubSpot apuntando a la URL de AF de uno de los registros duplicados en la tabla `tc_therapists`.

Así que cuando se accedió al AF del contacto incorrecto se cambió el valor del campo "Fecha de Vencimiento de la licencia de terapeuta" causando problemas de workflows.

## Datos


### Registro Principal 1️⃣

Current HS: `150239138865` - Older Record (2021)

- AF: `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`
- Estado:
	- Personal Information, Immunization, Info. Cred., Employment incompletos.
- Claves:
	- Es el contacto en uso. Tiene mezclado el secundario (abajo).
	- URL de AF en el contacto es `710d1d63-2eca-4860-b3cf-c492c0bb4810`
		- Apunta a otro registro

#### Valores en la BD

ID: `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`
email: `tanya16aug`
hubspot_id: `150239138865`
created_at: 2021-10-09
license exp. date: 2027-08-31
form_completed_at: 2021-10-18
latest AF at: null


### Registro Secundario (incorrecto) 2️⃣

Merged HS: `8380688778` - Newer Record (2024)

- AF: `710d1d63-2eca-4860-b3cf-c492c0bb4810`
- Estado: mucho más completo en todas las secciones
- Clave: la URL del AF es la que está en el contacto primario (arriba)

#### Valores en la BD

ID: `710d1d63-2eca-4860-b3cf-c492c0bb4810`
email: `tanya16aug+luna`
hubspot_id: `8380688778`
created_at: 2024-04-01
license exp. date: 2025-08-31
form_completed_at: 2024-04-09
latest AF at: nil



# Pasos para la solución

- Cambiar AF URL en el contacto principal a la que corresponde
	- terminarla en: `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`
- Reasociar immunization y prof history del contacto secundario
	- cambiar el tc_therapist_id
	- del valor `710d1d63-2eca-4860-b3cf-c492c0bb4810`
	- al valor `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`