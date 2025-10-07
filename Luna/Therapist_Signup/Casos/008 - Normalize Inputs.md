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

## Solución en backend

Claudio ideó un concern para reusar en los modelos que interesen. Hizo esto:
```ruby
module Normalizable
  extend ActiveSupport::Concern

  class_methods do
    def normalize_fields(*fields, type: :name)
      before_validation do
        fields.each do |field|
          value = send(field)
          next if value.blank?

          normalized = case type
                       when :name then normalize_name(value)
                       when :email then normalize_email(value)
                       else value
                       end

          # Assign the normalized value to the field. Notice the equals sign.
          send("#{field}=", normalized)
        end
      end
    end
  end

  private

  def normalize_name(value)
    value.strip.titleize
  end

  def normalize_email(value)
    value.strip.downcase
  end
end
```

Lo cual me parece muy útil.

## Solución Backfill

Estas son las rakes.

Backfill task:
```bash
bundle exec rake data:backfill_normalize_name_fields
```

Verification task:
```bash
bundle exec rake data:verify_normalize_name_fields
```

Usan queries SQL para hacer eliminar el whitespace y cambiar el case de los campos email.