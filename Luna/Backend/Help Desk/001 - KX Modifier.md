# 001 - KX Modifier error?

Tres casos reportados que no funcionan bien.

Voy a usar esta query o alguna variante de la misma para entender los datos de cada caso.
```sql
select
  pat.id as patient_id,
  epo.prompt_therapist_for_medical_necessity,
  mdts.effective_from,
  mdts.effective_until,
  mdts.threshold_exceeded,
  (case mcpmnr.medical_necessity_state
    when 0 then 'rejected'
    when 1 then 'approved'
  end) as medical_necessity_state
from patients pat
inner join episodes epo on epo.patient_id = pat.id
inner join medicare_dollar_threshold_statuses mdts on mdts.patient_id = pat.id
left join medicare_care_plan_medical_necessity_responses mcpmnr on mcpmnr.medicare_dollar_threshold_status_id = mdts.id
```

> [!Important]
> El campo `prompt_therapist_for_medical_necessity` en Episode es `true` por defecto. Solo eso no decide si se le muestra o no el prompt al Therapist en la app.
> 
> ```ruby
> t.boolean "prompt_therapist_for_medical_necessity", default: true, null: false
> ```

## Primer Caso

Reporte:
> A patient had a Medicare Prompt to Therapist selected but the system incorrectly displayed the Medicare Spending Limit Exceeded as No.

Esperado:
> Both the "Spending Limit Exceeded" and "Prompt Therapist for Medicare Medical Necessity" should be YES.

CP ID:
```
1f828722-5fb6-403b-8530-2ead14b5d87a
```

### Datos

La query arroja lo siguiente para 2025:

```
threshold_exceeded: false
prompt_therapist_for_medical_necessity: true
medical_necessity_state: NULL
```

Este care plan no tiene `medicare_care_plan_medical_necessity_responses`.

Así se refleja en Luxe:
![[001.case.01.luxe.png]]

### Respuesta

The value in "Prompt Therapist for Medicare Medical Necessity?" is always true so that once the Threshold is Exceeded the system can prompt the therapist the confirmation modal.

## Segundo Caso

Reporte:
> For patients that have Medicare Threshold approved by PT, *the submitted at date and Response date are the same*

Esperado:
> Response Time and Submitted At time should be two different times. Response Time should be when the PT responded and Submitted AT time should be when we initiated contact with the PT

CP ID:
```
f434cf88-009a-477b-bed9-e2323de0047a
```

### Datos

La query muestra esto para 2025:
```
response: 2025-08-23 05:49:27.848
submitted_at: 2025-08-23 05:49:27.848
```

Pasa porque en Luxe se usa el mismo valor `created_at` para ambas columnas.

Así se ve en Luxe:
![[001.case.02.luxe.png]]

### Respuesta

They report the same time because are using the same source value. It's confusing but the main point of the "Response" column is to report the rejected/approved state of the Medical Necessity.

## Tercer Caso

Reporte:
> For prompt request submitted by the system, the Submitted At date is missing

Esperado:
> If we submitted prompt to therapist- the Submitted At time should be stamped.

CP ID:
```
7ed82845-c9c2-4b6e-abcb-2696757107ac
```

### Datos

La query arroja lo siguiente para 2025:

```
threshold_exceeded: true
prompt_therapist_for_medical_necessity: true
medical_necessity_state: NULL
```

Así se ve en Luxe:
![[001.case.03.luxe.png]]

### Respuesta

The mobile app should prompt the therapist to confirm the medical necessity. For the momento there's no response so that's why there's no info in the "Medical Necessity Responses" section.