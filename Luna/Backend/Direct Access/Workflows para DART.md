# Workflows para DART

Estoy probando como crear workflows en HubSpot para poder hacer lo siguiente:

- Enviar webhook cuando haya nueva asociación a Contacto nuevo (🟡)
- Enviar webhook cuando un Credentialing se actualicé la propiedad `direct_access_restricted` (🟡)
- Enviar webhook cuando haya cambio de asociaciones entre Contact y Credentialing
- Enviar webhook cuando haya nueva asociación a Contacto existente

Lo clave de los workflows es que para que un Contact u objeto pase por ahí tiene que cumplir las reglas del enrollment. Estas reglas van a dictaminar si el objeto pasa la primera vez que existe o si puede ser re-enrolado.

## Workflow cuando hay nueva asociación Contacto nuevo

Armé el workflow de esta forma:

![[00.new.contact.workflow.png]]

En esta definición los Contacts pasaran por este workflow cuando:

- se les asocie un Credentialing object
- y la propiedad de Credentialing "Associated Contact Record Id" tenga un valor

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

> [!Note]
> Todo lo siguiente pasa teniendo en cuenta que el _Object Type_ del workflow es Contact.

Al hacer un segundo sign up o al asignar manualmente una asociación no se dispara este workflow.

Esto queda explicado por que no se cumplen las reglas de enrolamiento. Se puede ver en este mensaje de advertencia en las configuraciones del trigger del enrollment:

![[01.reenrollment.trigger.warning.png]]

La [documentación](https://knowledge.hubspot.com/workflows/add-re-enrollment-triggers-to-a-workflow#contact-based-workflows) al respecto dice que el re-enrollment no se da:

- cuando se usa una condición "_is any of, is equal to all of, and is equal to any of operators cannot be used for re-enrollment if they have multiple values_".
- si se usan propiedades de otro objeto: _You can only trigger re-enrollment using properties specific to the workflow object. For example, in a company-based workflow, you can only re-enroll based on company properties._

En la imagen de la configuración es un Workflow de Contact y estoy tratando de definir re-enrollment para una propiedad de otro objeto (Credentialing).

## Workflow cuando se actualice la propiedad `direct_access_restricted` de Credentialing

Este es el workflow que permitirá exponer este valor en los diferentes clientes. Es la parte crucial ya que esta propiedad solo se maneja desde HubSpot y tiene que reflejarse al día en backend.

Para lograr este Workflow tuve que usar un tipo diferente de Trigger. Para el workflow anterior se usa un trigger que se da cuando se cumplen condiciones predefinidas. En cambio, este workflow está basado en eventos en los datos del objeto (Contact, Credentialing).

![[03.event.based.workflow.trigger.png]]

> [!Note]
> La principal diferencia con este tipo de trigger es que permite definir condiciones que deben cumplirse para que empiece el workflow aparte de las condiciones que debe cumplir el objeto para que entre en este (enrollment).
>
> Tenemos así que podemos definir que empiece cuando se actualice una propiedad y que solo se enrolen Credentialings con label "Active Attested".
>
> ![[031.event.trigger.enrollment.png]]

Con esta forma de definir el workflow al activar el botón de re-enrollment no tenemos el warning que vemos en el Workflow anterior. Lo que significa que se puede enviar el webhook cada vez que hay un cambio.

### Propiedades que triggeran el workflow

Deberían ser:

- `direct_access_restricted`
	- Este es el flag como tal.
- `treating_license_state`
	- Tiene el estado de la Licencia.
- `therapist_name_state`
	- Es el nombre más el estado. Puede que cambie.

### Pruebas

Con la condición que "Direct Access Restricted" sea "_known_" pude probar recibir el webhook.

Eso lo pude probar en local una vez que active el workflow. En la primera prueba llegó esto:
```json
"label: active-attested"
"hs_object_id: 31006455396"
"associated_contact_record_id: 138861631756"
"label: active-attested"
```

En una prueba posterior luego de actualizar el workflow y el webhook llegó esto:
```json
"label: active-attested"
"hs_object_id: 31006455396"
"direct_access_restricted: Yes: OH < 2 years experience"
"associated_contact_record_id: 138861631756"
"label: active-attested"
```

Luego decidí cambiar la condición a que sea "_is any of_" ya que esta propiedad es una lista de valores. Así será más fácil captar los diferentes cambios que pueda tener.

```json
"label: active-attested"
"hs_object_id: 31006455396"
"direct_access_restricted: Yes: VA < 1 year experience no DPT - will need license"
"associated_contact_record_id: 138861631756"
"label: active-attested"
```