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

# Inspección

Con estas queries pude comprobar el estado de ambos registros.

Con esta encontré los dos registros. El original y duplicado.
```sql
select
  t.id,
  CONCAT(t.first_name,' ',t.last_name) as full_name,
  t.email,
  t.hubspot_id,
  t.registered_from,
  t.created_at,
  t.updated_at,
  t.physical_therapy_license_expiration_date,
  t.form_completed_at,
  t.latest_attestation_completed_at
from tc_therapists t
where t.email ilike '%tanya16aug%';
```

Con esta otra pude ver el estado de las asociaciones.
```sql
select
 t.id,
 CONCAT(t.first_name,' ',t.last_name) as full_name,
 tci.updated_at as cred_info_updated,
 ti.updated_at as immu_updated,
 tph.updated_at  as prof_updated,
 pref.updated_at as pref_updated,
 tnaca.updated_at as npi_updated,
 tp.updated_at as pay_updated
from tc_therapists t
left join tc_credentialing_informations tci on tci.tc_therapist_id = t.id
left join tc_immunizations ti on ti.tc_therapist_id = t.id
left join tc_professional_histories tph on tph.tc_therapist_id = t.id
left join tc_preferences pref on pref.tc_therapist_id = t.id
left join tc_npi_and_caqh_applications tnaca on tnaca.tc_therapist_id = t.id
left join tc_payouts tp on tp.tc_therapist_id = t.id
where t.email ilike '%tanya16aug%'
;
```

# Pasos para la solución

En HubSpot en el contacto Principal

- Cambiar el valor de la propiedad `physical_therapy_license_expiration_date` para que termine en: `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`
	- Este sería el ID que corresponde con el correo del contacto
- Cambiar el valor de la propiedad `credentialing_form_files` a `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`
- Reasociar immunization y prof history del contacto secundario
	- cambiar el tc_therapist_id
	- del valor `710d1d63-2eca-4860-b3cf-c492c0bb4810`
	- al valor `5dbf1a09-8feb-4df8-b8e3-4a5e6e2e71a0`


# Replicación

Ryan dijo esto cuando pedí la prod-op:
> patches should always be followed by root cause fixes, or at least investigation to determine scope of the fix (and a card to follow up on later, if cannot be fixed in a reasonable time now)

Por lo que creo que me toca investigar qué pasó aquí.

> [!Note]
> Docs de HubSpot sobre cómo mezclar registros -> https://knowledge.hubspot.com/records/merge-records
>
> Al mezclar, el primer contacto que se elige es el que se mantiene/primario. El segundo contacto se mezcla en el primario.

## Probando Mezclar Contactos

Veo que para este caso en las propiedades problemáticas el detalle en _source_ siempre dice "Record Merged". En cambio para otras interacciones dice "CRM UI" o "Integration".

Mi sospecha es que usaron la herramienta de Merge de HubSpot y eligieron el contacto sin tener en cuenta los valores necesarios.

Necesito comprobar que "Record Merge" solo se asigna cuando se usa la herramienta o se hace una petición al API.

### Intento Mezcla de Contactos

Primario: 161691502868
Secundario: 161671499930

Me dio error:
> Association configuration limits are exceeded in portal 7712148 when merging 161671499930 into 161691502868 of object type 0-1

El bot de HS dice que se está excediendo algún límite configurado de las asociaciones.

> [!Important]
> El error era que ambos contactos tenían un Credentialing en Active Attesting. Al cambiar el del secundario a otra label pude mezclar.