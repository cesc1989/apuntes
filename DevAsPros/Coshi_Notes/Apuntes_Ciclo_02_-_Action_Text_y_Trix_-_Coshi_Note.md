# Apuntes Ciclo 02 - Action Text y Trix - Coshi Notes

# Action Text y Trix

Seguí la [guía oficial de Rails](https://guides.rubyonrails.org/action_text_overview.html#installation)

    $ bin/rails action_text:install
          append  app/javascript/application.js
          append  config/importmap.rb
          create  app/assets/stylesheets/actiontext.css
    To use the Trix editor, you must require 'app/assets/stylesheets/actiontext.css' in your base stylesheet.
          create  app/views/active_storage/blobs/_blob.html.erb
          create  app/views/layouts/action_text/contents/_content.html.erb
    Ensure image_processing gem has been enabled so image uploads will work (remember to bundle!)
            gsub  Gemfile
           rails  railties:install:migrations FROM=active_storage,action_text
    Copied migration 20231230025702_create_action_text_tables.action_text.rb from action_text
          invoke  rspec
       identical    .rspec
           exist    spec
    

Al final modifica archivos de Rspec.

## Archivos y código generado

**Archivos**

- `app/assets/stylesheets/actiontext.css`
- `app/views/active_storage/blobs/_blob.html.erb`
- `app/views/layouts/action_text/contents/_content.html.erb`
- `db/migrate/20231230025702_create_action_text_tables.action_text.rb`
- `app/assets/stylesheets/trix.css`
    - Este lo tuve que descargar de [aquí](https://unpkg.com/trix@2.0.0/dist/trix.css).

**Código**
En `app/javascript/application.js`

    import "trix"
    import "@rails/actiontext"

En `config/importmap.rb`

    pin "trix"
    pin "@rails/actiontext", to: "actiontext.js"

**Código agregado por mí**
En pasos adicionales explicados en la guía oficial.

En `app/assets/stylesheets/application.scss`

    @import "./actiontext.css";

Para traer los estilos de actiontext y trix.

En `app/models/comment.rb`

    has_rich_text :trix_content

No tuve que agregar un nuevo campo.

En `app/views/comments/_comment.html.erb`

    <%= comment.trix_content %>

En `app/views/comments/_form.html.erb` el textarea pasó a:

    <%= f.rich_text_area :trix_content %>

En `app/controllers/comments_controller.rb` permito el nuevo atributo

    def comment_params
      params.require(:comment).permit(:plain_content, :theme_id, :trix_content)
    end


## Actualización 19 de Abril de 2024

Hay que agregar esta línea al archivo layout o en el caso de Puntapie en `_head.html.erb`

    <%= javascript_importmap_tags %>


# A tener en cuenta al usar Trix ‼️ 

**Los bloques de código son en texto plano. 🆗** 
Tal cual y pasa en Twist.

**Tiene un bug al escribir en móvil en Firefox en Android 9. ⚠️** 
No pasa en desktop ni en Chrome móvil. Lo reporté [aquí](https://github.com/basecamp/trix/issues/1116).

**Hay que actualizar estilos al escribir y al mostrar. 🆗** 
Cuando se escribe, estilos como los de cita, encabezado 1, enlace y código, no se muestran.

Pasa similar al mostrar el rich text en la vista. Hay que revisar y retocar estilos.

**Hay que ocultar botones mediante CSS. 🆗** 
No queda claro cómo configurar para quitar botones. Toca recurrir a ocultarlos por CSS.

**Faltan atajos de teclado.**
Por ejemplo, no hay para atravesar el texto.

**No tiene soporte para Markdown. ⚠️** 
Al escribir caracteres para generar negrita o cita, aparecen tal cual al mostrarse.

**Los enlaces abren en la misma pestaña. 🆗** 
Aunque se puede hacer con JavaScript para que abran en una pestaña aparte.

**El texto de Trix rompe la lista de Temas. ⚠️** 
No se cierra correctamente el HTML que contiene el texto rico generado por Trix y pasa lo de la captura.

![Trix rompe el HTML de la etiqueta <a>](https://paper-attachments.dropboxusercontent.com/s_6520D5BB5C95B5DCEF6C36B7B4638A6160655A9D72D6CE1405D2670A53A34789_1703979149823_broken.html.bc.trix.png)


**No hace bien los saltos de línea.** ⚠️ 
O hay que hacerlo doble o simplemente lo está haciendo mal.

![](https://paper-attachments.dropboxusercontent.com/s_6520D5BB5C95B5DCEF6C36B7B4638A6160655A9D72D6CE1405D2670A53A34789_1703979370431_imagen.png)


**¿Será una cosa de estilos CSS?**





# Trix y Stimulus

Tenía un controlador revisando los cambios de texto en el elemento `textarea` para habilitar/deshabilitar el botón “Guardar”.

Debido a que Trix usa su propio elemento HTML `trix-editor`,  este impide que funcione el código en Stimulus.

Por un lado, ese elemento `trix-editor` no escucha el evento “input” y el campo “hidden” no captura eventos porque no son enviados por un usuario.

![](https://paper-attachments.dropboxusercontent.com/s_6520D5BB5C95B5DCEF6C36B7B4638A6160655A9D72D6CE1405D2670A53A34789_1703912148772_Screenshot+2023-12-29+at+11.54.30+PM.png)



> Events are only triggered when the user performs the event in the browser, so if it's `<input type="hidden">` or an `<input>` hidden by CSS, the user won't be able to trigger events to your input.
> 
> [Stack Overflow](https://stackoverflow.com/a/1003155/1407371).


## ¿Cómo resolver esto?

Tengo dos alternativas:

1. Darle un `eventListener` al elemento trix-editor: ✅ 
2. Hacer un `trigger(``"``input``"``)` al campo oculto

Con la forma 1. fue que pude resolver.

Asigné el eventListener así:

    this.editor.addEventListener('input', this.toggleSubmit);

Y al final, en la función tuve que obtener el texto y encontrar el botón así:

    const contentValue = this.textContent;
    const saveButton = document.getElementById('save_comment');

El resto del código, funcionó.


# ¿Cómo agregar atajo de teclado en Trix?

Al parecer no hay forma de extender Trix para lograr esto pero usando JavaScript normal se podría logar.

Así lo expresa el autor Javan en este [comentario](https://github.com/basecamp/trix/issues/702#issuecomment-552503429) y este [otro](https://github.com/basecamp/trix/issues/643#issuecomment-505028254).

Ejemplos que pone:

    addEventListener("trix-initialize", event => {
      const { toolbarElement } = event.target
      const bulletButton = toolbarElement.querySelector("[data-trix-attribute=bullet]")
      bulletButton.setAttribute("data-trix-key", "u")
    })


Otro ejemplo:

    addEventListener("trix-initialize", event => {
      const { toolbarElement } = event.target
      console.log(toolbarElement.querySelectorAll("[data-trix-key]")
    })

Da como resultado:

    <button … data-trix-key="b">Bold</button>
    <button … data-trix-key="i">Italic</button>
    <button … data-trix-key="k">Link</button>
    <button … data-trix-key="z">Undo</button>
    <button … data-trix-key="shift+z">Redo</button>

Y alguien continua con un ejemplo de cómo lo hicieron para listas en [este comentario](https://github.com/basecamp/trix/issues/643#issuecomment-1848237135).

Para una sola letra funciona bien el atajo de teclado pero las combinaciones con SHIFT no están funcionando.


# Recursos y posts para extender Trix

Encontré varios artículos de los cuales puedo tomar ideas e inspiración para modificar y extender Trix.

Para tener en cuenta → https://www.betterstimulus.com/


- Customizing the Trix toolbar → https://matthaliski.com/blog/customizing-the-trix-toolbar/
- Hotwire: best practices for stimulus → https://dev.to/phawk/hotwire-best-practices-for-stimulus-40e
    - Busca el título MessageComposerController
- Add font-size controls to Trix's toolbar → https://dev.to/rockwell/add-font-size-controls-to-trixs-toolbar-1hgd
    - Tiene otros artículos enlazados del mismo autor


# Abrir los enlaces en una pestaña nueva

Trix no tiene una opción para que se guarde el atribut “target” al enlazar un texto. En varios [issues](https://github.com/basecamp/trix/issues/55) dejan claro que esto no lo van a poner en Trix. Lo bueno es que con un poco de [JavaScript](https://github.com/basecamp/trix/issues/55#issuecomment-1495234623) se soluciona.

Así cree el controlador para cambiar esto:

    import { Controller } from "@hotwired/stimulus"
    
    // Connects to data-controller="trix-links"
    export default class extends Controller {
      connect() {
        this.element.querySelectorAll('a').forEach(function(link) {
          if (link.host !== window.location.host) {
            link.target = "_blank";
          }
        })
      }
    }



