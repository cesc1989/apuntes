# Varios Apuntes para Attestation Form

# Sobre esta pregunta que le hice a Jose Han sobre los custom objects en Hubspot

**Q: Whenever a new contact is created, the related custom objects (credentialing, therapist address) are created automatically?**

*A: Indeed, the related custom objects are created automatically -- but not at the time of contact creation. (...)*

Los custom objects se crean cuando se cumplen las condiciones que se describen a continuación.

## Credentialing

**Credentialing custom object is ==Initial Form Date is known==.**

This means they *signed the contract*.

 The "Initial Form Date" value corresponds to the `signup_form_date` property internal name.  This value is set in the `HubspotContactService` class. This value is set right after they complete the Sign Up form at `/sign-up/` URL.

## Therapist Address

**Therapist Address custom object is ==created when "Initial Form Date" is known.==**

This means they *signed the contract*.

The "Initial Form Date" value corresponds to the `signup_form_date` property internal name.  This value is set in the `HubspotContactService` class. This value is set right after they complete the Sign Up form at `/sign-up/` URL.

## License

Main License object would be created by a Hubspot workflow.

Additional licenses will be created using HS API.

# 📞 Llamada del 4 de Septiembre, 2024

Se tocaron dos temas:
- Cómo se dispara el envío del Attestation Form
- Cómo se asegura que el Contact hace referencia al Therapist Address y Credentialing más reciente

### Cómo se dispara el envío del Attestation Form

En backend, se definirá un webhook para que desde Hubspot se haga una petición para enviar el correo con el enlace al Attestation Form.

Este webhook debo definirlo en el RFC con URL y parámetros. Una vez definido, compartir la info con Jose Han.

### Cómo se asegura que el Contact enlaza con el Therapist Address y el Credentialing más reciente

Un Contact podrá tener más de un Therapist Address. Para apunta al más reciente hay que encontrarlo vía una "Association Label". Hay dos definidas dentro del nuevo modelo de objetos en Hubspot.

- Current Address
- Relocation Address

Ambas tienen una relación 1-1 con Therapist Address.