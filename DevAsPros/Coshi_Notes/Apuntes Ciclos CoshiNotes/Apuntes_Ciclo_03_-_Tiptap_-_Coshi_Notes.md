# Apuntes Ciclo 03 - Tiptap - Coshi Notes

# Instalación y Configuración

Hay una sección oficial que explica cómo instalar con JS vainilla.

Enlaces:

- Instalación → https://tiptap.dev/docs/editor/installation/vanilla-javascript
- Configuración → https://tiptap.dev/docs/editor/guide/configuration


## Instalación con Stimulus

[+Apuntes Ciclo 02 - Tiptap - Coshi Notes](https://paper.dropbox.com/doc/Apuntes-Ciclo-02-Tiptap-Coshi-Notes-UKQhV7CDJIYANtISwj3Sl) 


# Tiptap StarterKit

Este paquete incluye [muchas extensiones](https://tiptap.dev/docs/editor/api/extensions/starter-kit). Las esenciales y otras adicionales para tener un editor funcional.

## Extensiones esenciales
- Document
- Paragraph
- Text
## Adicionales
- Heading
- History → [https://tiptap.dev/docs/editor/api/extensions/history](https://tiptap.dev/docs/editor/api/extensions/history)
- Dropcursor → [https://tiptap.dev/docs/editor/api/extensions/dropcursor](https://tiptap.dev/docs/editor/api/extensions/dropcursor)
- Gapcursor → [https://tiptap.dev/docs/editor/api/extensions/gapcursor](https://tiptap.dev/docs/editor/api/extensions/gapcursor)
- Placeholder → [https://tiptap.dev/docs/editor/api/extensions/placeholder](https://tiptap.dev/docs/editor/api/extensions/placeholder)


## Más Extensiones

Link → https://tiptap.dev/docs/editor/api/marks/link


# Limpiar contenido del Editor mediante botón y al hacer Submit

Tiptap ofrece un comando para eso [https://tiptap.dev/docs/editor/api/commands/clear-content](https://tiptap.dev/docs/editor/api/commands/clear-content)

    // Remove all content from the document
    editor.commands.clearContent()

Así lo hice con Stimulus.

En el botón descartar usé una acción de Stimulus.

    <%= button_tag 'Descartar', type: :button, class: 'btn btn-secondary', data: { action: 'rich-editor#clearContents' } %>

Controlador:

    clearContents() {
      this.element.editor.commands.clearContent()
    }

Y cuando el form hace Submit, agregué una línea adicional en la acción `formSubmit` que se creó en la instalación.


    formSubmit(event) {
      this.clearContents()
    }


# Bloquear y desbloquear el botón de “Guardar”

Encontré que tienen un [callback](https://tiptap.dev/docs/editor/api/events#update) `[#update](https://tiptap.dev/docs/editor/api/events#update)` y por ahí podría ser pero necesito encontrar el texto del editor para poder hacer la comparación y togglear el botón.

El texto lo obtengo con el [getter](https://tiptap.dev/docs/editor/api/editor#is-empty) `editor.isEmpty`.

Así lo hice para el uso normal.

    const $this = this
    
    const editor = new Editor({
      element: textArea,
      extensions: [
        StarterKit
        // (...)
      ],
      // (...)
      onUpdate({ editor }) {
        $this.toggleSubmitButton(editor.isEmpty)
      }
    })

Cuando carga el editor el botón está bloqueado. Al escribir, se desbloquea y cuando se limpia el contenido se bloquea. Sin embargo, si se escribe y se “Guarda” el botón se mantiene desbloqueado.

Para que se vuelva a bloquear, al terminar el submit, se vuelve a bloquear.

    formSubmit(event) {
      // (...)
      document.getElementById('submit-comment-btn').setAttribute('disabled', true)
    }

