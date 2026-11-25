# Lecciones y Aprendizajes #2

- Documentación de `form_with` [https://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_with](https://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_with)
- Ajax upload for files: https://github.com/JangoSteve/remotipart
- Cómo modificar el generador de Rails [https://stackoverflow.com/questions/8688220/rails-3-1-changing-default-scaffold-views-and-template](https://stackoverflow.com/questions/8688220/rails-3-1-changing-default-scaffold-views-and-template)


# Usando ActiveStorage
- Hay que correr el comando `rails active_storage:install`, para generar las migraciones de Active Storage
- luego, `rake db:migrate`. Notese que active storage funciona como una asociación polimorfica
- Para asignar un archivo a un modelo se puede con `has_one_attached :NAME` o `has_many_attached :NAMES`
- Para el caso de muchos, el input debe llevar la opción `multiple: true` para que así el parámetro aceptado por el controlador pueda ser un array `params.require(:owner).permit(documents: [])`
- Funciona normal con carga mediante Ajax sin tener que valerse de gemas
- Un muy [buen artículo](https://prograils.com/posts/rails-5-2-active-storage-new-approach-to-file-uploads) sobre ActiveStorage


## Previsualizar Documentos Adjuntos
> In Active Storage the previewable? method will return true for PDFs and Videos provided the system has the right binaries installed. The variable? method will return true for images if mini_magick is installed. If neither of these things is true then, the attachment is likely a file that is best viewed after being downloaded.

El método `#variable?` es para las imágenes que se carguen. Dará `true` siempre y cuando esté instalado la gema `[mini_magick](https://github.com/minimagick/minimagick)`:

- Acerca de la [previsualización de PDFs en Heroku](https://devcenter.heroku.com/articles/active-storage-on-heroku#attachment-previews)
- Puedo usar [MuPDF o Poppler](https://dzone.com/articles/rails-52-active-storage-previews-poppler-and-solvi) pero Poppler no da problemas de licencias
    - El [texto](https://blog.heroku.com/rails-active-storage) original
- Listado de [paquetes para Poppler](https://packages.ubuntu.com/source/trusty/poppler)
- Read the [script or watch](https://gorails.com/episodes/file-uploading-with-activestorage-rails-5-2) the video

**Cómo instalar Poppler en Ubuntu**

- [Ask Ubuntu](https://askubuntu.com/questions/781552/how-to-install-the-latest-version-of-poppler)
- [Stack Overflow](https://stackoverflow.com/questions/32156047/how-to-install-poppler-in-ubuntu-15-04)

`**ActiveStorage**` **docs**

- `[ActiveStorage::Blob::Representable](https://api.rubyonrails.org/classes/ActiveStorage/Blob/Representable.html)` docs
- `[ActiveStorage::Blob](https://edgeapi.rubyonrails.org/classes/ActiveStorage/Blob.html#method-i-filename)`
- `[ActiveStorage::Filename](https://edgeapi.rubyonrails.org/classes/ActiveStorage/Filename.html)`
# Cosas de Turbolinks

Al desplegar uno de los menús de la barra lateral e ir a una página, cuando se quiere volver a desplegar, no lo hace. Esto ocurría por turbolinks. [Cómo desbloquear](https://codkal.com/rails-how-to-remove-turbolinks/).

# ¿Se puede agregar campos adicionales a la tabla generada por Active Storage?

La idea sería poder agregar un campo, por ejemplo, tipo, a la tabla `active_storage_attachments` para así poder buscar un determinado adjunto.

La otra opción es crear una tabla `documents` y ahí utilizar `ActiveStorage` pero...

En Stack Overflow sugieren usar una tabla para los adjuntos y ahí usar el macro de active storage pero implica que haya tres tablas:

- tabla principal
- tabla de adjuntos
- tabla de active storage

por lo que hay que tener en cuenta las consultas N+1. Ver: `active_storage_attachments`

Fijate en el método `[with_attached_photos](https://api.rubyonrails.org/classes/ActiveStorage/Attached/Macros.html#method-i-has_many_attached)` de ActiveStorage. Y así es cómo implementan el macro de clase `[with_attached_x](https://github.com/rails/rails/blob/master/activestorage/lib/active_storage/attached/model.rb#L52)`.

# `nested forms` junto a `simple form`
- [Wiki](https://github.com/plataformatec/simple_form/wiki/Nested-Models)
- [Readme](https://github.com/plataformatec/simple_form#simple-fields-for)
- y que no se te olvide [el contexto](https://stackoverflow.com/a/18436806/1407371) del formulario
- nested form y simple form [se llevan bien](https://github.com/ryanb/nested_form#simpleform-and-formtastic-support)
# Quitar valor por defecto de un campo con migración

Se usa la migración `change_column` y se le da valor por defecto, *nil*

    class RemoveDefaultFromKindInReference < ActiveRecord::Migration[5.2]
      def change
        change_column :references, :kind, :integer, default: nil
      end
    end

**Nota**: No olvidar especificar el mismo tipo de campo.

# ¿Cuál es la diferencia entre `include_blank` y `prompt` en el helper *select* en Rails?

Ver [Stack Overflow](https://stackoverflow.com/questions/11302728/whats-the-difference-between-include-blank-and-prompt-in-the-rails-select-hel#11304099).

Según la respuesta y comentarios `include_blank: true` mostrará una opción vacía (o sea que se puede elegir un valor en blanco).

En cambio `prompt: true` ejecuta:

    I18n.translate('helpers.select.prompt', :default => 'Please select')

Si hay una relación seleccionada, entonces `prompt` no mostrará la opción en blanco. Por su parte `include_blank` siempre mostrará una opción en blanco para elegir.


# Slim y el detalle de espacios o tabuladores

Hay que tener cuidado con Slim que siempre sean espacios y no tabs. Al copiar y pegar puede que vengan espacios demás:


    thead
    -tr
    --th
    # ^ will work
    
    thead
    ..tr
    ....th
    # ^ so will this
    
    thead
    ..tr
    -..th
    # ^ will produce malformed alignment error

Visto en [Stack Overflow](https://stackoverflow.com/questions/9013500/ruby-slim-format-for-table). [Respuesta](https://stackoverflow.com/a/42290482/1407371) (nota mi comentario)

