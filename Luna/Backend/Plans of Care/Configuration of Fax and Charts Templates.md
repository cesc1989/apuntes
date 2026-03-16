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