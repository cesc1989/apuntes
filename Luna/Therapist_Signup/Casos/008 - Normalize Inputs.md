# 008 - Normalize Inputs

Etiquetas: #luna_help_desk 

Caso EDG-2505

Pidieron que los campos de nombres y similares sean limpiados antes de ser guardados en la BD. Hagamos esto quitando espacios y convirtiendo el case a minúsculas o tipo nombre.

## Campos a Normalizar

No todos los campos necesitan ser normalizados. Voy a empezar con los que la gente puede escribir cosas como nombres propios o de empresas.

**Therapist (app/models/therapist.rb)**

- first_name
- last_name
- email
- middle_name
- maiden_name
- entity_name (company name)
- employer_name

**PersonalReference (app/models/personal_reference.rb)**

- full_name
- email
- title (job title)

**Alias (app/models/alias.rb)**

- name (alias names - should be treated like person names)

**WorkRecord (app/models/work_record.rb)**

- employer_name
- employer_type
- employer_email (trim + lowercase)
