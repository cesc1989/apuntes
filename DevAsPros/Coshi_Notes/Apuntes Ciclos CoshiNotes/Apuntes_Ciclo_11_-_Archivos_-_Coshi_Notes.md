# Apuntes Ciclo 11 - Archivos - Coshi Notes
En este ciclo fue cuando completamos todo lo faltante para salirnos de Twist.

# Librerías para visualizar imágenes en Galería

Encontré estas por ahora:

- PhotoSwipe https://photoswipe.com/getting-started/
- Bootstrap Lightbox https://github.com/trvswgnr/bs5-lightbox


# Cómo hacer que Capybara presione enter?

Para emular el funcionamiento de la caja de búsqueda, la cual envía la petición al presionar “enter”, en Capybara se hace así:

    "cadena\n"

Justo después del string de prueba se escribe el carácter para salto de línea.

    find_field(class: 'search-input').fill_in(with: "Gohan\n")

Visto [https://stackoverflow.com/a/75769604/1407371](https://stackoverflow.com/a/75769604/1407371)


# Carga de archivos con Active Storage

Cuando se usa `has_many_attached :x` hay que usar el atributo “multiple:true” en el input file

    <%= f.file_field :documents,
              multiple: true,
              accept: "application/pdf",
              class: 'd-none',
              data: { rich_editor_target: "fileExplorer" } %>

Sin eso, el form no creará el campo con el valor adecuado en “name” para que lo reciba el controlador.

Cuando quiera hacer lo de borrar adjunto de manera individual, en [este artículo](https://nicholasshirley.com/several-strategies-to-delete-activestorage-attachments/) muestran un poco sobre eso.

Finalmente, aquí están los callback en versión after_commit [https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit)

Normalmente los callbacks serían:

    after_commit :do_foo_bar, on: [:create, :update]

pero Rails da unos atajos en la forma:

    after_create_commit(*args, &block)
    after_destroy_commit(*args, &block)

y esos son los que veo en los tutoriales de Turbo Stream.


# Stimulus Controller: Copia de enlace cuando está en elemento oculto

Cuando pasé el botón “Copiar enlace del comentario” al dropdown, el controlador de Stimulus conectaba bien pero cuando clicaba el botón la acción no se ejecutaba. Lo que hacía era conectarse de nuevo.

Se puede ver en el [commit correspondiente](https://github.com/cesc1989/coshinotes/commit/25415238fbe1b7bac86adca4d7d4159fd3cde7fa) el cambio que tuve que hacer.

El error fue que había dejado el data-controller en un padre incorrecto.

![](https://paper-attachments.dropboxusercontent.com/s_275D08F4F6C528DF5D537DC76B36E6C6A792DEBE62F55C5C791787DE83CDD78D_1705930303626_copy.contr.error.png)



