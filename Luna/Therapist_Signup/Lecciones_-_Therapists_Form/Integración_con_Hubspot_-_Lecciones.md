# Integración con HubSpot - Lecciones

# Sobre el formato de los campos `Date` y `DateTime` para HubSpot

Hubspot [almacena los valores para estos tipo de campos en milisegundos](https://developers.hubspot.com/docs/faq/how-should-timestamps-be-formatted-for-hubspots-apis). En [esta respuesta en la comunidad](https://integrate.hubspot.com/t/valid-formats-for-date-fields-when-posting-via-hubspot-api/507/2) encontré una forma de poder generar dicho *timestamp* para lo que pide Hubspot:

```ruby
def hubspot_formatted_date(date)
	return nil if date.nil?

	Time.utc(date.year, date.month, date.day).strftime('%s%3N')
end
```

El detalle es que tiene que ir en mili segundos pero el tiempo tiene que referirse a la media noche de la fecha.

Como Hubspot no guarda la hora, lo que hace es almacenar la fecha con las horas cero, minutos cero y segundos cero de la media noche en UTC.

> HubSpot has two types of CRM object properties for storing times, date and datetime.  Date picker properties created in HubSpot in would be date properties, so would store only the date and **not** the time.
> 
> Date properties will only store the date, and must be set to midnight UTC for the date you want.  For example, May 1 2015 would be 1430438400000 (01 May 2015 00:00:00 UTC). If you try to set a value that is not midnight UTC, you will receive an error. In HubSpot, date properties always display the specific date they are set to, regardless of the time zone setting of the portal or the user.

## 2025: Docs de HubSpot sobre campos Date y Datetime

Ver aquí -> https://developers.hubspot.com/docs/guides/api/crm/properties

Para Date dice:
> A value representing a specific day, month, and year. Values must be represented in UTC time and can be formatted as [ISO 8601 strings or EPOCH-timestamps in milliseconds](https://developers.hubspot.com/docs/guides/api/crm/properties#add-values-to-date-and-datetime-properties) (i.e. midnight UTC).

Para Datetime dice:
> A value representing a specific day, month, year, and time of day. Values must be represented in UTC time and can be formatted as [ISO 8601 strings or UNIX-timestamps in milliseconds](https://developers.hubspot.com/docs/guides/api/crm/properties#add-values-to-date-and-datetime-properties).

Las descripciones enlazan a [Add values to date and datetime properties](https://developers.hubspot.com/docs/guides/api/crm/properties#add-values-to-date-and-datetime-properties). En esa sección explican más cómo debe ser el guardar valores date o datetime.

## Error al enviar un DateTime a una propiedad Date en HubSpot

Intenté enviar un DateTime al campo `signup_form_date` y obtuve este error:
```bash
Hubspot::RequestError (Response body: {"validationResults":[{"isValid":false,"message":"1744557443000 is at 15:17:23.0 UTC, not midnight!","error":"INVALID_DATE","name":"signup_form_date"}],"status":"error","message":"Property values were not valid","correlationId":"d806ba1d-bcba-4339-891d-c27ce99b29f4"}):
```

Queda claro que los campos tipo Date solo reciben la fecha con la hora en midnight (00:00:00).

# Sobre las Propiedades de Contactos en HubSpot

> Contact properties are used to store specific information for each of your contact records. **HubSpot has several default contact properties**, but **you can also create custom properties to store whatever data you need** for your contact records. Properties are used as the criteria for contact lists, to segment your contacts, and also as the starting criteria for workflows.

[See documentation](https://developers.hubspot.com/docs/methods/contacts/contact-properties-overview).

En resumen, hay unas propiedades por defecto que ya Hubspot provee:

- *firstname*
- *lastname*
- *phone*
- *zip*
- *email*
- *address*
- *city*
- *state*
- *date_of_birth*

Y otras que se pueden agregar, sin embargo, esas que se agregan tienen que ser agregadas en un grupo de propiedades. El flujo es así:

- [Se crea el grupo de propiedades](https://developers.hubspot.com/docs/methods/contacts/v2/create_contacts_property_group)
- [Se crea(n) la(s) propiedad(es)](https://developers.hubspot.com/docs/methods/contacts/v2/create_contacts_property)
- [Se crea el contacto](https://developers.hubspot.com/docs/methods/contacts/create_contact)

# Cómo guardar los valores para preguntas en HubSpot que son checkboxes

Se explica en [este FAQ](https://developers.hubspot.com/docs/faq/how-do-i-set-multiple-values-for-checkbox-properties):

> When setting multiple values for a checkbox property, the values should be separated with a semicolon (;).
> 'value1;value2;value5'

Ejemplo para una petición:
```ruby
{
	"property": "property_name",
	"value": "value1;value3;value4"
}
```

