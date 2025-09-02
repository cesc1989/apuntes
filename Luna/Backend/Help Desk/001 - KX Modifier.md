# 001 - KX Modifier error?

Tres casos reportados que no funcionan bien.

## Primer Caso

Reporte:
> A patient had a Medicare Prompt to Therapist selected but the system incorrectly displayed the Medicare Spending Limit Exceeded as No.

Esperado:
> Both the "Spending Limit Exceeded" and "Prompt Therapist for Medicare Medical Necessity" should be YES.

CP ID:
```
1f828722-5fb6-403b-8530-2ead14b5d87a
```

## Segundo Caso

Reporte:
> For patients that have Medicare Threshold approved by PT, the submitted at date and Response date are the same

Esperado:
> Response Time and Submitted At time should be two different times. Response Time should be when the PT responded and Submitted AT time should be when we initiated contact with the PT

CP ID:
```
f434cf88-009a-477b-bed9-e2323de0047a
```

## Tercer Caso

Reporte:
> For prompt request submitted by the system, the Submitted At date is missing

Esperado:
> If we submitted prompt to therapist- the Submitted At time should be stamped.

CP ID:
```
7ed82845-c9c2-4b6e-abcb-2696757107ac
```