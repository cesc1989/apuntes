# Workflows para DART

Estoy probando como crear workflows en HubSpot para poder hacer lo siguiente:

- Enviar webhook cuando un Credentialing se actualicé la propiedad `direct_access_restricted`  
- Enviar webhook cuando haya cambio de asociaciones entre Contact y Credentialing
- Enviar webhook cuando haya nueva asociación a Contacto existente
- Enviar webhook cuando haya nueva asociación a Contacto nuevo

Lo clave de los workflows es que para que un Contact u objeto pase por ahí tiene que cumplir las reglas del enrollment. Estas reglas van a dictaminar si el objeto pasa la primera vez que existe o si puede ser re-enrolado.

## Workflow cuando hay nueva asociación Contacto nuevo

Definí un workflow de esta forma:

![[00.new.contact.workflow.png]]

En esta definición, los Contactos pasaran por este workflow cuando:

- se les asocie un Credentialing object
- y la propiedad "Associated Contact Record Id" tenga un valor

Una vez entren en el workflow se decantan en una de las cuatro ramas que corresponden con las cuatro etiquetas de asociación existentes:

- Active Attested
- Active
- Inactive
- Processing for Move

### Pruebas

Usando Ngrok como la URL para el webhook pude comprobar que:

- Al activar el workflow, una vez HS calcula cuantos objetos pueden ser enrolados, envía una petición al webhook por cada Contact según cumplas las condiciones antes mencionadas.
- Al crear un nuevo Contact a los pocos minutos que existe con su Credentialing se envía la petición al webhook.

#### Associated Contact Record ID vacío en webhook

Noté que en varias peticiones del webhook llegaba el cuerpo sin un valor en la propiedad `associated_contact_record_id`. Ejemplo:

```json
"label: active-attested"
"state: CA"
"hs_object_id: 31006455396"
"hs_createdate: 1752778944619"
"direct_access_state: "
"therapist_name_state: Coshinita Kuintero - California"
"direct_access_restricted: "
"associated_contact_record_id: "
"label: active-attested"
```

Esto podría ser un delay al momento de enviar el webhook. Al revisar a posteriori el objeto Credentialing se puede apreciar que tiene un valor en dicha propiedad.

#### Reasociar Credentialings no dispara el workflow

Al hacer un segundo sign up o al asignar manualmente una asociación no se dispara este workflow.

Esto queda explicado por que no se cumplen las reglas de enrolamiento. Se puede ver en este mensaje de advertencia en las configuraciones del trigger del enrollment:

![[01.reenrollment.trigger.warning.png]]

La [documentación](https://knowledge.hubspot.com/workflows/add-re-enrollment-triggers-to-a-workflow#contact-based-workflows) al respecto dice que cuando se usa una condición "_is any of_, _is equal to all of_, and _is equal to any of_ operators cannot be used for re-enrollment if they have multiple values_".

No me queda claro si la regla que puse en el workflow cabe aquí pero por las pruebas para que sí lo es.