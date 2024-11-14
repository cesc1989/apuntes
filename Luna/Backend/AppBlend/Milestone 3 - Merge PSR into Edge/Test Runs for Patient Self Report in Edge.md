# Tests runs for Patient Self Report in Edge

Estos son todos los tests para correr de Patient Self Report una vez mezclados en Edge. Pongo este listado aquí para que sea más fácil de encontrar y tener los comandos a la mano.

## Carpeta por Carpeta

En orden según la carpeta `spec`:
```bash
pruebas spec/forms/patient_self_report/

pruebas spec/lib/patient_self_report/
pruebas spec/lib/section_builder/
pruebas spec/lib/webhooks/completed_form_spec.rb

pruebas spec/models/patient_self_report/

pruebas spec/requests/patient_self_report/api/v1/
pruebas spec/requests/patient_self_report/api/v2/
pruebas spec/requests/patient_self_report/api/v3/

pruebas spec/routing/patient_self_report/

pruebas spec/serializers/patient_self_report/

pruebas spec/services/patient_self_report/
```

## En Total

Todos en una sola línea:
```bash
pruebas spec/forms/patient_self_report/ spec/lib/patient_self_report/ spec/lib/section_builder/ spec/lib/webhooks/completed_form_spec.rb spec/models/patient_self_report/ spec/requests/patient_self_report/api/v1/ spec/requests/patient_self_report/api/v2/ spec/requests/patient_self_report/api/v3/ spec/routing/patient_self_report/ spec/serializers/patient_self_report/ spec/services/patient_self_report/
```

## Patrón "form" en paralelo

```bash
be rake "parallel:spec[form]"
```