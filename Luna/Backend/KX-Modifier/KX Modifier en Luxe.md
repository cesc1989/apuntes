# KX Modifier en Luxe - Patient / Care Plan views

Estas son las cosas que se muestran en Luxe en las páginas de detalle de Care Plan y de Paciente. Además del form de edit de Paciente y Care Plan.

## En Care Plan Detail

Si hay *Medicare Dollar Threshold Status*:

Se muestra una casilla con "No" cuando `threshold_exceeded` es `false`. Y "Yes" en caso contrario.

![threshold not exceeded](./attachments/201.limit.ok.png)

Si está excedido el threshold y existe un *Medicare Care Plan Medical Necessity Response*:

- Muestra la respuesta si fue *approved* o *rejected*.
- Muestra qué Therapist respondió al Medical Necessity.

![threshold exceeded](./attachments/202.limit.exceeded.medical.necessity.png)

Si está excedido el threshold y no existe un *Medicare Care Plan Medical Necessity Response*, muestra un mensaje que indica que se le mostrará el Prompt al Therapist.

![show prompt](./attachments/203.limit.exceeded.prompt.png)

## En Care Plan Edit

Se muestra un checkbox para marcar si se muestra o no el Prompt.

![checkbox for prompting therapist](./attachments/204.care_plan.prompt.png)

## En Patient Profile

Se muestra una vista igual a la del Care Plan detail.

![patient care plan detail from profile](./attachments/207.patient.care_plan.rejected.necessity.png)

## En Patient Edit

Cuando solo hay *Medicare Dollar Threshold Status* aparece información de las fechas de cobertura y el checkbox para marcarlo como excedido o no.

![patient edit form](./attachments/205.patient.edit.medicare.status.png)

Cuando hay *Medicare Dollar Threshold Status* y *Medicare Care Plan Medical Necessity Response*, aparecen campos para cambiar los valores de este último.

![patient edit form medical necessity](./attachments/206.patient.edit.medicare.necessity.png)