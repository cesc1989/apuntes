# MedID

## ¿Qué es un MedID?

Un MedID es un **identificador opaco de string** (ej: `"O7Lkgum8REe51ORohZQXq4EflJFWGuMl"`) asignado por el servicio externo **OMFS/MedPicker** que identifica de forma única una combinación específica de:

> **(Medicamento + Proveedor + Farmacia)**

En la API de MedPicker se llama `remoteProductIdentifierNk` (Remote Product Identifier Natural Key). No son UUIDs — son strings opacos de tipo base64.

---

## Dónde aparece en Rails

| Modelo | Campo | Significado |
|--------|-------|-------------|
| `Salesforce::Medication` | `medid__c` / alias `med_id` | **Primary key** — clave principal del registro de medicamento importado de MedPicker |
| `Ontraport::Script` | `med_id_sent` | MedID que se envió a MedPicker al momento de la orden (lo que el médico quiso prescribir) |
| `Ontraport::Script` | `med_id_prescribed` | MedID que fue realmente prescrito por CareValidate (lo que se escribió en el Rx) |
| Payloads JSON | `"MedId"` / `"medId"` | Clave en hashes de recomendaciones que fluyen entre componentes del workflow de prescripción |

Los dos campos del script pueden **diferir** — `med_id_prescribed` puede quedar desincronizado del snapshot original (`script.med_id_prescribed` can drift from it).

---

## Sinónimos en el codebase

| Nombre | Contexto |
|--------|----------|
| `med_id` | Alias Ruby en `Salesforce::Medication`, `Ontraport::Script`, `RxWrittenBundle::Recommendation` |
| `medid__c` | Columna raw de Salesforce / Heroku Connect en la tabla `medication` |
| `medId` / `MedId` | Payloads JSON (webhook Beluga, hashes de medpicker_recommendations) |
| `med_id_sent` | Campo del script de Ontraport — MedID al momento de la orden |
| `med_id_prescribed` | Campo del script de Ontraport — MedID al momento de escribir el Rx |
| `remoteProductIdentifierNk` | Clave JSON de la API de MedPicker |
| `remote_product_identifier_nk` | Versión snake_case en modelos `Omfs::` y `ApiData::ProviderScriptProduct` |

---

## Flujo principal

```
MedPicker API → remoteProductIdentifierNk
      ↓
  med_id_sent  (al firmar up / hacer orden)
      ↓
  [CareValidate decide]
      ↓
  med_id_prescribed  (Rx escrito)
      ↓
  Salesforce::Medication.find_by!(med_id:)  →  MedicationRequest
```

---

## Módulos y clases clave

- **`Medpicker` module** (`app/models/medpicker.rb`) — fachada para buscar datos de MedPicker por MedID. Provee `get_fulfillment_data(med_id:, provider_id:)` y helpers derivados (`get_base_product_name`, `get_weeks_supply`, `get_rx_strength`, `get_remote_quantity`, `get_display_name`). Lanza `Medpicker::ProductNotFound` cuando `remoteProductIdentifierNk` está en blanco.
- **`Omfs::MedPickerApi`** — 3 endpoints REST:
  - `get_med_id` → `/api/MedPicker/get-medid`
  - `get_med_id_beluga` → `/api/MedPicker/Beluga/get-medid`
  - `get_med_id_care_validate` → `/api/MedPicker/CareValidate/get-medid`
- **`Omfs::MedIdRequestCommand`** — wraps `{ medId:, providerId: }` para las llamadas REST
- **`CareValidate::Validation::KnownMedIds`** — valida que el MedID sea reconocido por MedPicker
- **`CareValidate::Validation::PrescribedMedIdExists`** — verifica que el MedID del payload de decisión resuelva a un `remoteProductIdentifierNk` válido
- **`CareValidate::Validation::BaseMedMatch`** — compara el producto + tipo de dosis del MedID recomendado vs. el prescrito
- **`Salesforce::MedpickerImporter`** — importa datos de MedPicker y crea/actualiza registros `Salesforce::Medication` usando el MedID como clave
- **`Salesforce::MedicationRequestRecording`** — hace `Salesforce::Medication.find_by!(med_id:)` antes de crear un `Salesforce::MedicationRequest`. Mantiene una lista `KNOWN_SUPPLY_MED_IDS` (5 MedIDs de suministros como jeringas) que se omiten silenciosamente.

---

## Sentinel value

`Salesforce::Medication::NULL_MEDICATION_ID = "NULL_MEDICATION"` — valor centinela para referencias faltantes a `ProviderScriptProduct`.

---

## Resumen

MedID es el **"SKU externo"** de un producto farmacéutico tal como lo conoce MedPicker. Es el hilo que conecta el sistema interno de órdenes con el sistema externo de dispensación. No identifica un medicamento genérico, sino una combinación específica de medicamento + proveedor + farmacia.
