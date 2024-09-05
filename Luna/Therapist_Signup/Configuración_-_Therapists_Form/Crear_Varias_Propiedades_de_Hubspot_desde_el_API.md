# Crear Varias Propiedades de Hubspot desde el API

El formulario desde la web es bien lento para crear propiedades y tampoco permite crearlas en bloque.

Los groupName son:

    # Dev
    credentialing
    
    # Alpha
    therapist_credentialing

Con este llamado a la API se pueden crear en grupo. [Documentación](https://developers.hubspot.com/docs/api/crm/properties).

```bash
   curl --request POST \
     --url 'https://api.hubapi.com/crm/v3/properties/contact/batch/create' \
      --header 'authorization: Bearer TOKEN' \
      --header 'content-type: application/json' \
      --data '{
      "inputs": [
        {
          "name": "do_you_have_any_of_the_following_symptoms_",
          "groupName": "credentialing",
          "hidden": false,
          "displayOrder": 2,
          "label": "do_you_have_any_of_the_following_symptoms_",
          "hasUniqueValue": false,
          "type": "string",
          "fieldType": "text",
          "formField": true
        }
      ]
    }'
```

Para campos tipo fecha usar

    {
      "type": "date",
      "fieldType": "date"
    }


## Los Permisos para Access Token de private app

El access token necesita estos permisos para poder hacer peticiones REST

![[properties-permissions.png]]

[Docs](https://developers.hubspot.com/docs/api/crm/properties) busca "Create a batch of properties”.

# Para cada entorno

Para Development
```bash
 curl --request POST \
      --url 'https://api.hubapi.com/crm/v3/properties/contact/batch/create' \
      --header 'authorization: Bearer TOKEN' \
      --header 'content-type: application/json' \
      --data '{
      "inputs": [
        {
          "name": "physical_therapy_license_initial_date",
          "groupName": "credentialing",
          "hidden": false,
          "displayOrder": 2,
          "label": "physical_therapy_license_initial_date",
          "hasUniqueValue": false,
          "type": "date",
          "fieldType": "date",
          "formField": true
        }
      ]
    }'
```

Para Alpha
```bash
curl --request POST \
      --url 'https://api.hubapi.com/crm/v3/properties/contact/batch/create' \
      --header 'authorization: Bearer TOKEN' \
      --header 'content-type: application/json' \
      --data '{
      "inputs": [
        {
          "name": "physical_therapy_license_initial_date",
          "groupName": "therapist_credentialing",
          "hidden": false,
          "displayOrder": 2,
          "label": "physical_therapy_license_initial_date",
          "hasUniqueValue": false,
          "type": "date",
          "fieldType": "date",
          "formField": true
        }
      ]
    }'
```


# Crear Contactos

Docs → https://developers.hubspot.com/docs/api/crm/contacts#create-contacts

```bash
curl --request POST \
      --url 'https://api.hubapi.com/crm/v3/objects/contacts/batch/create' \
      --header 'authorization: Bearer TOKEN' \
      --header 'content-type: application/json' \
      --data '{
      "inputs": [
        {
          "properties": {
            "email": "elcoshinita@gmail.com",
            "firstname": "Coshiampirejo",
            "lastname": "Super Coshi"
          }
        }
      ]
    }'
```