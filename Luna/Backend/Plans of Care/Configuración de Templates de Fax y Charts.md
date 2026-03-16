# Configuración de Templates de Fax y Charts

Todo gira alrededor de la constante `GOOGLE_DOC_TEMPLATE_MAP`. Esta se define en dos clases:

- `PlansOfCare::PlanOfCareFaxPdfService`
- `Charts::GoogleDocsService`

Esta constante define los IDs de los templates para cada Fax según el Payer y de cada Chart según los estados.

La clase `PlansOfCare::PlanOfCareFaxPdfService` configura un hash de estas claves con su respectivo ID que corresponde a un documento de Google Docs.

```ruby
[
 :medicare_initial,
 :medicare_standard,
 :medicare_progress,
 :medicare_discharge,
 :direct_access_initial,
 :direct_access_standard,
 :direct_access_progress,
 :direct_access_discharge,
 :fax_on_demand_initial,
 :fax_on_demand_standard,
 :fax_on_demand_progress,
 :fax_on_demand_discharge,
 :referred_initial,
 :referred_standard,
 :referred_progress,
 :referred_discharge,
 :payer_initial,
 :payer_standard,
 :payer_progress,
 :payer_discharge,
 :workers_comp_initial,
 :workers_comp_standard,
 :workers_comp_progress,
 :workers_comp_discharge,
 :temp_storage_folder_id
]
```

Mientras que la clase `Charts::GoogleDocsService` configura estas:
```ruby
[:unsigned_chart, :signed_chart, :patient_chart_summary, :temp_storage_folder_id]
```

## Asignación de ID de Template por llave del hash

Hay templates que se reusan en varias llaves. A continuación emparejo cada template con las llaves que deben usar su ID.

### Direct Access Fax

Llaves:
```
direct_access_initial
direct_access_standard
direct_access_progress
```

### Discharge Fax

Llaves:
```
medicare_discharge
direct_access_discharge
referred_discharge
payer_discharge
```

### Generic POC Fax

Llaves:
```
fax_on_demand_initial
fax_on_demand_standard
fax_on_demand_progress
fax_on_demand_discharge
```

### Medicare Non Standard Fax

Llaves:
```
medicare_initial
medicare_progress
```

### Payer Fax

Llaves:
```
payer_initial
payer_standard
payer_progress
```

### Referred / Standard Fax

Llaves:
```
referred_initial
referred_standard
referred_progress
medicare_standard
```

### Workers Compensation Fax

Llaves:
```
workers_comp_initial
workers_comp_standard
workers_comp_progress
workers_comp_discharge
```

### Patient Chart Summary

Llaves:
```
patient_chart_summary
```

### Signed Chart

Llaves:
```
signed_chart
```

### Unsigned Chart

Llaves:
```
unsigned_chart
```

## Folder Temporal

La generación de los charts y pocs incluye una carpeta Temporal.

La llave es `temp_storage_folder_id`. Se usa en ambas clases que definen `GOOGLE_DOC_TEMPLATE_MAP`.

## En la base de datos

Existe un registro en `Setting` que guarda todas estas llaves. Como las clases definen unos ids quemados en el código si se quiere usar unos diferentes toca guardarlos en la bd.

Los dejo aquí porque si genero un nuevo dump de alpha voy a perder los valores.

### Actualizar Setting key `google_template_ids`

Así actualizo con los IDs correspondiente a mi configuración local:
```ruby
Setting.find_by!(key: "google_template_ids").update!(
	value: {
		unsigned_chart: "13gxNyBdwxpRqNWcCYiBlQ2JtbPVW5GggNHmOIjglyWI",
		signed_chart: "1ppQfyFR8UMO9ZAN989tx5SYkW6dhj17PQbTIZEaIW1k",
		patient_chart_summary: "1Ohetc-kMWrOvs3K9tNBtKNDNQ-qsJ4kXITleA8fLJsU",
		medicare_initial: "1In5aG9hNY43xmS7PAzuPfHljVgUfKJvLNDKM-3-vSJg",
		medicare_standard: "17thBPUNzrJiRtdB48CeJuuz7Hq6dD568UtRUj7RCYA8",
		medicare_progress: "1In5aG9hNY43xmS7PAzuPfHljVgUfKJvLNDKM-3-vSJg",
		medicare_discharge: "1DZ8hbBfW2rGwN-gP4O0nVYoL8S6nnRYeFC46Vsxz3F4",
		direct_access_initial: "1uQRjCcZlce0L6jFLbmuHaMiBmzvK6j53HNjjWRKLi9M",
		direct_access_standard: "1uQRjCcZlce0L6jFLbmuHaMiBmzvK6j53HNjjWRKLi9M",
		direct_access_progress: "1uQRjCcZlce0L6jFLbmuHaMiBmzvK6j53HNjjWRKLi9M",
		direct_access_discharge: "1DZ8hbBfW2rGwN-gP4O0nVYoL8S6nnRYeFC46Vsxz3F4",
		fax_on_demand_initial: "15MMemcJSHC1VK8GAT1hbyQGtjkHRh9z1qAVBGranGis",
		fax_on_demand_standard: "15MMemcJSHC1VK8GAT1hbyQGtjkHRh9z1qAVBGranGis",
		fax_on_demand_progress: "15MMemcJSHC1VK8GAT1hbyQGtjkHRh9z1qAVBGranGis",
		fax_on_demand_discharge: "15MMemcJSHC1VK8GAT1hbyQGtjkHRh9z1qAVBGranGis",
		referred_initial: "17thBPUNzrJiRtdB48CeJuuz7Hq6dD568UtRUj7RCYA8",
		referred_standard: "17thBPUNzrJiRtdB48CeJuuz7Hq6dD568UtRUj7RCYA8",
		referred_progress: "17thBPUNzrJiRtdB48CeJuuz7Hq6dD568UtRUj7RCYA8",
		referred_discharge: "1DZ8hbBfW2rGwN-gP4O0nVYoL8S6nnRYeFC46Vsxz3F4",
		payer_initial: "1xDMZzefp6nxJPjvgsIjipQHVNeXuSAi9ebCqoF32lHo",
		payer_standard: "1xDMZzefp6nxJPjvgsIjipQHVNeXuSAi9ebCqoF32lHo",
		payer_progress: "1xDMZzefp6nxJPjvgsIjipQHVNeXuSAi9ebCqoF32lHo",
		payer_discharge: "1DZ8hbBfW2rGwN-gP4O0nVYoL8S6nnRYeFC46Vsxz3F4",
		workers_comp_initial: "1rhJInTX9K5uuzmPn6dQXEQ_fmRPazaZFO9uvYRmDIRY",
		workers_comp_standard: "1rhJInTX9K5uuzmPn6dQXEQ_fmRPazaZFO9uvYRmDIRY",
		workers_comp_progress: "1rhJInTX9K5uuzmPn6dQXEQ_fmRPazaZFO9uvYRmDIRY",
		workers_comp_discharge: "1rhJInTX9K5uuzmPn6dQXEQ_fmRPazaZFO9uvYRmDIRY",
		temp_storage_folder_id: "1o9wAllP1nohGarE7PMSCbmX2T3WYYqz_"
	}.to_json
)
```

### Revisar Setting key `google_template_ids`

Y así lo extraigo para comparar:
```ruby
Setting.load_safe("google_template_ids")
```