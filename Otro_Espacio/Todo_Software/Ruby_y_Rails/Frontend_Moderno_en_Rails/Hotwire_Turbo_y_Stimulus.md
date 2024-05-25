# Hotwire: Turbo y Stimulus
Enlaces variados:

- [Comunidad de Hotwire](https://discuss.hotwire.dev/)
- [Drifiting Ruby](https://www.driftingruby.com/episodes?free=true&pro=false&query%5Bname%5D=&tag=hotwire)
- [Go Rails](https://gorails.com/episodes/hotwire-rails?autoplay=1) (gratis)
- [Go Rails](https://gorails.com/series/stimulus-js) (Pago)
- [Turbo Showcase](https://turbo-showcase.herokuapp.com/)
- [Hover cards with Stimulus and Hotwire](https://boringrails.com/articles/hovercards-stimulus/)
- [Awesome StimulusJS](https://github.com/skatkov/awesome-stimulusjs)
    - Lista de artículos, tutoriales y paquetes hechos con Stimulus
## ¿En dónde usar Hotwire?

Lo que creo que los casos de uso más adecuados según lo que he probado son:

- Buscar y listar sin recargar la página
- Filtros y listar resultados sin recargar la página
- Enviar formularios y reflejar el cambio en los registros
- Efectuar validaciones del formulario sin refrescar la página y al vuelo
- Editar registros sin ir a un formulario en otra página
- Recibir actualizaciones de canales diferentes
    - Todo lo que asemeje una conversación: chat, mensajería
- Gráficas interactivas


## Turbo Rails Helpers

En el vídeo tutorial de Go Rails veo que Chris usa helpers como:

- `turbo_stream_from`
- `turbo_frame_tag`

Sin embargo, en la documentación de Turbo se mencionan son etiquetas:

- `<turbo-frame>`
- `<turbo-stream>`

Así que me nació la pregunta: ¿de dónde los sacaron? Pues resulta que salen de la [gema turbo-rails](https://github.com/hotwired/turbo-rails/tree/main/app/helpers/turbo).

También usó en el modelo los siguientes helpers para los callbacks:

    after_create_commit { broadcast_prepend_to "tweets" }
    after_update_commit { broadcast_replace_to "tweets" }
    after_destroy_commit { broadcast_remove_to "tweets" }

Sobre los métodos `broadcast_X_to` la documentación la veo en el código fuente de [turbo-rails](https://github.com/hotwired/turbo-rails/blob/main/app/models/concerns/turbo/broadcastable.rb) también. [Documentación](https://rubydoc.info/github/hotwired/turbo-rails/Turbo/Broadcastable/ClassMethods) sobre los métodos.

## Sobre las gemas turbo-rails, stimulus-rails y hotwire-rails

**La gema** [**turbo-rails**](https://github.com/hotwired/turbo-rails/)
Al instalarla y ejecutar el comando `rails turbo:install`, el paquete [NPM Turbo](https://github.com/hotwired/turbo) se agregará al `package.json` y se instalará también.

También añade el *concern* [Turbo::Broadcastable](https://github.com/hotwired/turbo-rails/blob/main/app/models/concerns/turbo/broadcastable.rb) que incluye helpers de ayuda para hacer *broadcasting* hacía las vistas.

**La gema** [**stimulus-rails**](https://github.com/hotwired/stimulus-rails)
Al instalarla y ejecutar el comando `rails stimulus:install`, se instala el paquete [NPM Stimulus](https://github.com/hotwired/stimulus) y se configuran algunos archivos JS.

**La gema** [**hotwire-rails**](https://github.com/hotwired/hotwire-rails)
~~Una forma conveniente de instalar las dos gemas anteriores al ejecutar el comando~~ `~~rails hotwire:install~~`~~.~~
Ya esta gema la depreciaron porque no aportaba valor.

# Los Pilares de Turbo

Turbo empaca varias técnicas para crear aplicaciones web modernas sin usar demasiado JavaScript.

Con Turbo, le permites al servidor servir HTML directamente de manera que toda la lógica puede mantenerse en el lenguaje de programación (en este caso, Ruby). La lógica queda en el servidor y el navegador solo le compete el HTML.

## Turbo Drive

Usa el mismo modelo de proceso-persistente que usan las SPAs pero sin un enrutador del lado del cliente ni estado global que manejar. Lo hace interceptando todos los clics en los `<a href>` del mismo dominio. Lo que hace Turbo Drive es impedir al navegador seguir el enlace, cambia la URL del navegador usando la [History API](https://developer.mozilla.org/en-US/docs/Web/API/History), pide la siguiente página usando un [fetch](https://developer.mozilla.org/en-US/docs/Web/API/fetch), y luego dibuja la respuesta HTML.

Aunque se puede interactuar directamente con Turbo Drive para controlar cómo trabaja, la mayoría de las veces es solo instalarlo y dejar que haga lo suyo.

> Turbo Drive es lo que era Turbolinks.

**Desactivar Turbo en enlaces o formularios**
Se logra usando el atributo `<data-turbo="false">`

    <a href="/" data-turbo="false">Disabled</a>
    
    <form action="/messages" method="post" data-turbo="false">
      ...
    </form>

De esta forma el navegador se encargará normalmente de ellos.

**Recargando los assets al ser modificados**
Se logra con el atributo `<data-turbo-track=``"``reload``"``>`:

    <head>
      ...
      <link rel="stylesheet" href="/application-258e88d.css" data-turbo-track="reload">
      <script src="/application-cbd3cd4.js" data-turbo-track="reload"></script>
    </head>


## Turbo Frames

Permite actualizar secciones dentro de una página que son independientes la una de la otra. Por ejemplo, en un foro de discusión la barra de navegación, lista de mensajes, un formulario, una lista de temas en un menú lateral son elementos independientes y se pueden cargar y actualizar de manera independiente.

Al usar Turbo Frames, se pueden envolver esos elementos dentro de marco (frame) para limitar su alcance de navegación y ser cargado de manera diferida (lazy loading). Al limitar su alcance de navegación toda la interacción se mantiene dentro del marco: clicar enlaces, enviar formularios.

De esa forma, se mantiene el resto de la página sin alterar. Para envolver una sección en un marco, se usa la etiqueta `<turbo-frame>`:

    <turbo-frame id="new_message">
      <form action="/messages" method="post">
        ...
      </form>
    </turbo-frame>

Podría sonar a los `<iframe>`s normales pero no lo son. Los Turbo Frames son parte del mismo DOM, estilizados por el mismo CSS, hacen parte del mismo contexto de JavaScript y no están bajo ninguna restricción de seguridad.

Entre otras ventajas están: Cacheo eficiente, ejecución en paralelo y listo para web móvil.

## Turbo Streams

Permiten cambiar cualquier parte de la página en respuesta a actualizaciones enviadas mediante una conexión WebSocket.

Turbo Stream brinda la etiqueta `<turbo-stream>` con cinco acciones básicas: *append*, *prepend*, *replace*, *update* y *remove*. Junsto a estas acciones se usa la propiedad `target` para especificar el ID del elemento sobre el cual se harán las mutaciones.

**Streamea respuestas HTTP**
Lo que hacía en el proyecto Leisure Shelf Playground para mostrar los errores de validación mediante un stream está dado por el MIME type `text/vnd.turbo-stream.html`.

    def destroy
      respond_to do |format|
        format.turbo_stream { render turbo_stream: turbo_stream.remove(@message) }
        format.html { redirect_to messages_url }
      end
    end

Pasa que Turbo sabe que debe pegar elementos `<turbo-stream>` cuando llegan en la respuesta de un formulario con el MIME type anterior. Cuando se envía un formulario cuyo método es cualquiera de POST, PUT, PATCH o DELETE Turbo inyecta al mime type en el grupo de formatos de respuesta que están en la cabecera Accept.

**Reusando Templates HTML**
Con Turbo, no necesitas serializar la respuesta en un JSON. La respuesta de Turbo puede ser un partial normal.

Por ejemplo, este parcial:

    <%= render partial: "messages/message", collection: @messages %>

En la respuesta del controlador se puede reusar así:

    class MessagesController < ApplicationController
      def create
        respond_to do |format|
          format.turbo_stream do
            render turbo_stream: turbo_stream.append(:messages, partial: "messages/message", locals: { message: message })
          end
    
          format.html { redirect_to messages_url }
        end
      end
    end


## Turbo Native

Las ventajas de Turbo para aplicaciones móviles híbridas.

# Los Pilares de Stimulus

Según la [documentación](https://stimulus.hotwire.dev/handbook/origin#the-three-core-concepts-in-stimulus):

    <div data-controller="clipboard">
      PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
      <button data-action="clipboard#copy">Copy to Clipboard</button>
    </div>
> Stimulus doesn’t bother itself with creating the HTML. Rather, it attaches itself to an existing HTML document. The HTML is, in the majority of cases, rendered on the server either on the page load (first hit or via Turbo) or via an Ajax request that changes the DOM.
> 
> Stimulus is concerned with manipulating this existing HTML document. Sometimes that means adding a CSS class that hides an element or animates it or highlights it.

O sea que Stimulus es como un reemplazo más moderno para jQuery.


> This makes Stimulus very different from the majority of contemporary JavaScript frameworks. Almost all are focused on turning JSON into DOM elements via a template language of some sort. Many use these frameworks to birth an empty page, which is then filled exclusively with elements created through this JSON-to-template rendering.
> 
> Stimulus also differs on the question of state. Most frameworks have ways of maintaining state within JavaScript objects, and then render HTML based on that state. Stimulus is the exact opposite. State is stored in the HTML, so that controllers can be discarded between page changes, but still reinitialize as they were when the cached HTML appears again.


## Entendiendo Stimulus

Lo que viene a continuación son apuntes tomados de leer la documentación y de probar usando Stimulus.

**Controladores**
La base de Stimulus es conectar elementos del DOM con objetos JavaScript. Estos objetos se llaman **controladores**.

> Nota: sin conectar un elemento del DOM a un controlador, no hay forma de ejecutar las acciones del código JS.

El controlador saca su nombre del nombre del archivo obviando el sufijo `_controller.js`. Ejemplos:

    hello_controller.js -> hello
    form_toggle_controller.js -> form-toggle
> Para entender los nombres de controladores, [ver esta sección](https://stimulus.hotwire.dev/handbook/installing#controller-filenames-map-to-identifiers).

Stimulus provee la función `connect()` la cual se ejecuta cada vez que un controlador se logra enlazar con un elemento del DOM.

    import { Controller } from "stimulus"
    
    export defautl class extends Controller {
      connect() {
        console.log("hola mundo", this.element)
      }
    }

**Acciones**
Son los métodos de los controladores que reciben los eventos del DOM. Se definen usando el atributo `data-action` en el elemento DOM que puede recibir el evento.

    <div data-controller="hello">
      <input type="text">
      <button data-action="click->hello#greet">Greet</button>
    </div>

**Targets**
[Docs](https://stimulus.hotwire.dev/handbook/hello-stimulus#targets-map-important-elements-to-controller-properties).
Es la forma en que podemos referenciar elementos a las propiedades del controlador.

Se logra la referencia también usando el atributo `data` pero con una variación. Se escribe el nombre del controlador entre las palabras data y target:

    <div data-controller="hello">
      <input data-hello-target="name" type="text">
      <button data-action="click->hello#greet">Greet</button>
    </div>

Para terminar el enlace con el controlador, hay que definir la propiedad en el mismo.

    import { Controller } from "stimulus"
    
    export default class extends Controller {
      static targets = [ "name" ]
    
      greet() {
        const element = this.nameTarget
        const name = element.value
        console.log(`Hello, ${name}!`)
      }
    }
> Nota como Stimulus crea una propiedad llamada `nameTarget`. Por cada valor en la lista de targets, Stimulus creará una propiedad con la forma `variableTarget`.


