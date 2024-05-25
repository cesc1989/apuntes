# Apuntes Ciclo 02 - Tiptap - Coshi Notes

# Configurando Tiptap

Seguí la [guía oficial](https://tiptap.dev/docs/editor/installation/vanilla-javascript) y este [otro tutorial](https://maxencemalbois.medium.com/migrating-from-trix-to-tiptap-in-a-rails-7-app-with-turbo-and-stimulus-js-97f253d13d0) para integrarlo en Rails con Stimulus.

Estos fueron los comandos para instalar los paquetes con Importmap

    $ ./bin/importmap pin @tiptap/core
    Pinning "@tiptap/core" to https://ga.jspm.io/npm:@tiptap/core@2.1.13/dist/index.js
    Pinning "@tiptap/pm/commands" to https://ga.jspm.io/npm:@tiptap/pm@2.1.13/commands/dist/index.js
    (...)
    
    $ ./bin/importmap pin @tiptap/starter-kit
    Pinning "@tiptap/starter-kit" to https://ga.jspm.io/npm:@tiptap/starter-kit@2.1.13/dist/index.js
    Pinning "@tiptap/core" to https://ga.jspm.io/npm:@tiptap/core@2.1.13/dist/index.js
    Pinning "@tiptap/extension-blockquote" to https://ga.jspm.io/npm:@tiptap/extension-blockquote@2.1.13/dist/index.js

El Starter Kit incluye varias extensiones básicas como negrita e italica. [Lista completa](https://github.com/ueberdosis/tiptap/blob/develop/packages/starter-kit/package.json#L31-L50).

        "@tiptap/core": "^2.2.0-rc.7",
        "@tiptap/extension-blockquote": "^2.2.0-rc.7",
        "@tiptap/extension-bold": "^2.2.0-rc.7",
        "@tiptap/extension-bullet-list": "^2.2.0-rc.7",
        "@tiptap/extension-code": "^2.2.0-rc.7",
        "@tiptap/extension-code-block": "^2.2.0-rc.7",
        "@tiptap/extension-document": "^2.2.0-rc.7",
        "@tiptap/extension-dropcursor": "^2.2.0-rc.7",
        "@tiptap/extension-gapcursor": "^2.2.0-rc.7",
        "@tiptap/extension-hard-break": "^2.2.0-rc.7",
        "@tiptap/extension-heading": "^2.2.0-rc.7",
        "@tiptap/extension-history": "^2.2.0-rc.7",
        "@tiptap/extension-horizontal-rule": "^2.2.0-rc.7",
        "@tiptap/extension-italic": "^2.2.0-rc.7",
        "@tiptap/extension-list-item": "^2.2.0-rc.7",
        "@tiptap/extension-ordered-list": "^2.2.0-rc.7",
        "@tiptap/extension-paragraph": "^2.2.0-rc.7",
        "@tiptap/extension-strike": "^2.2.0-rc.7",
        "@tiptap/extension-text": "^2.2.0-rc.7"

Agregar los paquetes mediante Importmap agregó un montón de líneas al archivo `config/importmap.rb`.


## Tiptap y Rails

Tiptap no funciona sobre un elemento textarea sino que toca darle el comportamiento a un div normal.

Para poder guardar el texto, hay que pasarle el contenido al campo del modelo y luego sí enviar el formulario:

    formSubmit(event) {
        event.preventDefault()
        event.stopPropagation()
    
        const formField = this.element.querySelector(".tiptap__field-content")
    
        formField.value = this.element.editor.getHTML()
        Turbo.navigator.submitForm(this.element)
    }

Lo ideal es tener una estructura de HTML que envuelva al menú de herramientas y al editor, además del campo oculto para el formulario.

    <div class="tiptap_editor">
        <div class="tiptap_toolbar">
            <div class="tiptap__button-group">
              <button type="button" class="tiptap__button tiptap__button--bold" data-action="click->resumeDataTiptap#bold">B</button>
             </div>
         </div>
    
          <div class="editor"></div>
          <%= f.hidden_field :plain_content, class: "tiptap__field-content" %>
      </div>
    </div>

Y así se configuró Tiptap:

    import { Controller } from "@hotwired/stimulus"
    
    import { Editor } from '@tiptap/core'
    import StarterKit from '@tiptap/starter-kit'
    
    initTiptap(masterElement) {
        const tiptapEditor = masterElement.querySelector(".tiptap_editor");
        const editor = new Editor({
          element: textArea,
          extensions: [
            StarterKit,
          ],
          content: 'Responder...',
          autofocus: true,
          editable: true,
          injectCSS: false
        });
    
        masterElement.editor = editor;
    }


Finalmente, para mostrar el contenido con HTML, hay que pasarlo como HTML seguro con el helper de Rails

    <%= (comment.plain_content || "").html_safe %>

Y ha funcionado bastante bien.


# A tener en cuenta al usar Tiptap ‼️ 

**Hay que configurar todos los botones del menú. ⚠️** 
No incluye ningún menú como Trix. Toca crear el menú, darle estilos y luego darle comportamientos mediante JavaScript.

*Importante: No necesito un toolbar en Coshi Notes. El editor de Twist no muestra ninguno. Necesito más los atajos de teclado.*

**Hay que actualizar estilos al escribir y al mostrar. 🆗** 
Similar a Trix, muchos elementos quedan sin estilos como código, citas, listas, etc.

**Sí tiene soporte para markdown. 🆗** 
Pero solo al escribir.

**Hay que agregar extensiones para todo lo que no sea texto básico**. ⚠️ 
Por ejemplo, para los enlaces hay que agregar la extensión Link.

**Hay que reajustar comportamiento para limpiar campo o desactivar botones.** ⚠️
Para limpiar el área de texto hay que cambiar el código que buscaba el textarea y lo limpiaba.

Lo mismo para habilitar/deshabilitar el botón “Guardar”.

