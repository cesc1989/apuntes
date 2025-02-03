# Mapeo de Secciones y Propiedades License Object

Complementario a [[Credentialing Object - Mapeo de Secciones y Propiedades]]

# 1 - Signup

| Field Name               | HS Property Name        |
|--------------------------|-------------------------|
| License Number           | license_number          |
| License Expiration Date  | license_expiration_date |

**Notas**

Cuando el usuario completa el registro en `/sign-up` Hubspot creará un objeto License. Sin embargo, la asociación no se da con el Contact sino a un Credentialing.

¿Hay que enviar estos valores en luego de completar el signup?

# 2 - Credentialing Information

| Field Name               | HS Property Name        | Fuente        |
|--------------------------|-------------------------|---------------|
| License Issue Date       | license_issue_date      | credentialing |
| License Issuing State    | license_issuing_state   | licenses      |
| License Number           | license_number          | licenses      |
| License Expiration Date  | license_expiration_date | licenses      |

**Notas**

*licenses* hace referencia a los registros de la tabla `licenses` asociada a `credentialing_informations`.

En el Credentialing Application, en la sección "Information for Credentialing" hay la opción para agregar una o más licencias en otros estados. El custom object License hace referencia a esos registros creados.