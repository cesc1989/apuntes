# Apuntes Ciclo 04 - Coshi Notes - Despliegue
Documento relacionado: [+Apuntes Ciclo 04 - Post Despliegue](https://paper.dropbox.com/doc/Apuntes-Ciclo-04-Post-Despliegue-Mk9QTxW3u8uyFGQmXCWZ0) 

Página para encontrar íconos e imágenes gratis → https://www.flaticon.com/search?type=icon&word=notebook


# Importante Despliegue

Tocó crear la carpeta `vendor` en `~/coshinotes/app/` porque al rsync fallaba sin esta y tampoco la copiaba a pesar de estar en el repo.

## Instalación de Redis

Para que Turbo Stream funcione necesita Redis instalado. Lo instalé siguiendo [esta guía](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04#step-1-installing-and-configuring-redis). No me convenció la instalación mediante paquete de Snap.

    sudo apt-get update
    sudo apt-get install -y redis-server


# Cargar contenido existente en el editor Tiptap para hacer la edición

Una vez ya implementada la edición, necesitaba montar el texto HTML del div al editor.

Para lograrlo Tiptap ofrece algunos métodos como [setContent](https://tiptap.dev/docs/editor/api/commands/set-content), sin embargo, en la documentación explican que eso es lo mismo que dar el texto al iniciar el Editor. Así que lo hice de esa forma.


> It’s basically the same as setting the `content` on initialization.

Quedó así:

    initTiptap(masterElement) {
      // (...)
    
      // Carga el texto cuando se edita el comentario
      const formField = this.element.querySelector(".tiptap__field-content")
      const editableContent = formField.value ? formField.value : null
    
      const editor = new Editor({
        // (...)
        content: editableContent,
      })
    }


# Atajos de Teclado con Stimulus

Encontré algunos proyectos pero no los usé:

- Stimulus Hotkeys → https://github.com/leastbad/stimulus-hotkeys
- Hotkeys-JS → https://github.com/jaywcjlove/hotkeys-js


Para lograr el atajo de teclado para hacer CMD + Enter y guardar el comentario al final lo logré así:

    import { Controller } from "@hotwired/stimulus"
    
    // Connects to data-controller="hotkeys"
    export default class extends Controller {
      static targets = ["submitCommentButton"]
    
      submitCommentButtonShortcut(event) {
        let keyCode = event.keyCode;
        let ctrlKey = event.ctrlKey;
        let metaKey = event.metaKey;
    
        if (metaKey && keyCode == 13) {
          const submitBtn = document.getElementById('submit-comment-btn')
          submitBtn.click()
        }
      }
    }

Así configuré el form:

    <%= form_with(
      model: comment,
      local: true,
      data: { controller: 'rich-editor hotkeys' },
      id: dom_id(comment)
    ) do |f| %>

Y así configuré el botón:

    <%= f.submit 'Guardar', id: 'submit-comment-btn', class: 'btn btn-success', disabled: true, data: { hotkeys_target: 'submitCommentButton' } %>

Además, también había que poner una acción en el editor para poder detectar presionar teclas.

Tocó hacerlo mediante JavaScript ya que la configuración del editor se hace desde Stimulus.

    // app/javascript/controllers/rich_editor_controller.js
    connect() {
      const masterElement = this.element;
    
      /*
        Programmatically adding a Stimulus submit action to the form
        in order to populate form field before submit.
      */
      const actions = "submit->rich-editor#formSubmit keydown->hotkeys#submitCommentButtonShortcut"
      masterElement.setAttribute("data-action", actions)
      this.initTiptap(masterElement);
    }

Enlaces adicionales:

- En los docs de Stimulus explican más sobre los [eventos de teclado](https://stimulus.hotwired.dev/reference/actions#keyboardevent-filter).
- Las extensiones de Tiptap [ya traen](https://tiptap.dev/docs/editor/api/keyboard-shortcuts) atajos de teclado. 



# Error con asset en image_tag

Me salió este error al abrir la página luego del primer despliegue:

    ActionView::Template::Error
    The asset "home_page" is not present in the asset pipeline.
    
    Sprockets::Rails::Helper::AssetNotFound
    The asset "home_page" is not present in the asset pipeline.

Resulta que los assets en `image_tag` deben referenciar la extensión. Si bien funciona sin esta en desarrollo, para producción es mejor que esté presente.

Visto aquí [https://stackoverflow.com/a/72986667/1407371](https://stackoverflow.com/a/72986667/1407371)

