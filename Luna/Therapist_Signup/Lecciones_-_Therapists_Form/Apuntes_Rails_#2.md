# Apuntes Rails #2

# Cómo listar los campos de un modelo?

Lo podemos hacer con `ModelName.column_names`:

    Therapist.column_names
    => ["id",
     "first_name",
     "phone_number",
     "email",
     "zip_code",
    ]

y de esta forma se puede obtener una lista de campos requeridos:

    Therapist.validators.grep(ActiveRecord::Validations::PresenceValidator).flat_map(&:attributes)
    => [:first_name,
     :last_name,
     :phone_number,
     :treating_postal_code,
    ]

Visto en [SO](https://stackoverflow.com/a/58612420/1407371).


# Rails 7 cambia to_s(format) a to_fs(format)

En Rails 7 se depreciarion varias cosas (deprecated).

Antes tenía esto:

    to_s(:luna)

A esto:

    to_fs(:luna)

Enlaces:

- [PR que introduce](https://github.com/rails/rails/pull/43772) el cambio
- En este [post describen](https://hayford.dev/this-week-in-rails-wrapped-an-overview-of-rails-7-1-features-part-i/#09-to_fsformat-replaced-to_sformat) varios cambios de Rails 7. Incluido este.
- [Documentación de to_fs](https://api.rubyonrails.org/classes/DateTime.html#method-i-to_formatted_s).
# Rail 7 render ya no acepta ivars

Otra cosa que fue depreciada (deprecated)

El mensaje de depreciación

    DEPRECATION WARNING: Passing instance variables to `render` is deprecated.
    In Rails 7.1, @immunization will be ignored.
     (called from generate_pdf at /Users/francisco/projects/luna-project/therapist-credentialing-backend/app/uploaders/tuberculosis_screening_pdf_uploader.rb:33)

Así estaba el controlador:

    @medicare_requirement
    
    format.pdf do
      render(
        pdf: 'medicare_requirement',
        show_as_html: params.key?('debug'),
        layout: 'pdf.html'
      )
    end

Así quedó luego de cambiar a locals

    render(
      pdf: 'medicare_requirement',
      show_as_html: params.key?('debug'),
      layout: 'pdf',
      locals: { medicare_requirement: @medicare_requirement }
    )

Y así era en el Uploader:

        WickedPdf.new.pdf_from_string(
          ac.render_to_string(
            template: 'immunizations/show',
            formats: [:pdf],
            layout: 'pdf',
            locals: { :@immunization => @immunization }
          )
        )

Y quedó así:

        WickedPdf.new.pdf_from_string(
          ac.render_to_string(
            template: 'immunizations/show',
            formats: [:pdf],
            layout: 'pdf',
            locals: { immunization: @immunization }
          )
        )


# Comparando strings de forma sencilla

En vez de comparar si una string tiene un texto con `include?` lo podemos hacer más eficiente con `casecmp`.

Doc de casecmp → https://ruby-doc.org/core-3.1.0/String.html#method-i-casecmp

Ejemplo que tengo en `PersonalInformationForm`:

    def city_as_other?
      city_of_birth&.casecmp('other')&.zero?
    end

Lo cual pude haber escrito como:

    city_of_birth.downcase.include?('other')

La diferencia es que casecmp retorna: -1, 0, 1 o nil. Y lo explican así:


- -1 if `other_string.downcase` is larger.
- 0 if the two are equal.
- 1 if `other_string.downcase` is smaller.
- `nil` if the two are incomparable.

Entonces sí la comparación con casecmp arroja cero quiere decir que hay una coincidencia.

