# Summary Page: Detalles para JSON Components

# Detalles

Cuando se carga el summary_page, los siguientes endpoints se les hace petición:

    /v1/diseases
    /v1/pain_spots
    /v1/summaries
    /v1/intake_forms

Los endpoints `/diseases` y `/pain_spots` son para listas las opciones de las secciones *Past Medical History* y *Body Parts*.

Por su parte, `/summaries` devuelve los datos para poder pintar:

- La información del paciente
- La lista de intake + progress forms. La respuesta incluye:
    - la URL para hacer la petición al endpoint `/form_results`
    - la URL para hacer la petición al endpoint `/intake_forms`


![](https://paper-attachments.dropboxusercontent.com/s_E08E2C25D12D79739E8975F522D90D2B871401AAD11260B6CECF4DF96BA90D83_1718750137813_001.summary.page.top.png)


Cuando se hace clic en cualquier “Outcome Measure” se llama a `/v1/form_results`, además del endpoint `/v1/questions`.

Cuando se hace clic en “Medical History”, el frontend reusa la respuesta de la primera llamada al endpoint `/v1/intake_forms`.

