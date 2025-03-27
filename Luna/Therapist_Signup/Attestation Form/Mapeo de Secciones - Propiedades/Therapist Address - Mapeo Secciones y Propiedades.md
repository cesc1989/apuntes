# Mapeo Secciones y Propiedades de Therapist Address Object

Complementa [[Credentialing Object - Mapeo de Secciones y Propiedades]] y [[License Object - Mapeo de Secciones y Propiedades]]

# 1 - Preferences

| Field Name    | HS Property Name | Fuente     |
|---------------|------------------|------------|
| Street Line 1 | street_line_1    | preference |
| State         | state            | preference |
| City          | city             | preference |
| Zip Code      | zip_code         | therapist  |

**Notas**

- Para obtener el zip_code de therapist se manda el mensaje `postal_or_zip_code`.