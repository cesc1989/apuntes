# Lecciones y Aprendizajes #3

- Eventos de ActiveStorage [https://guides.rubyonrails.org/active_storage_overview.html#direct-upload-javascript-events](https://guides.rubyonrails.org/active_storage_overview.html#direct-upload-javascript-events)
## Cómo obtener los *event listeners* *attachados* a un objeto DOM
- Chrome Dev Tools: https://developers.google.com/web/tools/chrome-devtools/console/command-line-reference#geteventlistenersobject
- SO: https://stackoverflow.com/questions/446892/how-to-find-event-listeners-on-a-dom-node-when-debugging-or-from-the-javascript#3426352


## ¿A quién se le adjunta el event listener cuando se usa de la forma `addEventListener()`?

Al objeto *window* https://stackoverflow.com/questions/38186271/when-one-calls-addeventlistener-without-a-target-element-what-element-does-it-de


## Cómo limpiar un input file luego que se termine la carga:
- SO: https://stackoverflow.com/questions/1043957/clearing-input-type-file-using-jquery#13351234


## Cargar fuente de una imagen desde el contenido de un *input file*
- SO: https://stackoverflow.com/questions/3814231/loading-an-image-to-a-img-from-input-file
- Como lo hace *dropify*:
    - Línea 268: https://github.com/JeremyFagis/dropify/blob/master/src/js/dropify.js#L268
    - Línea 166: https://github.com/JeremyFagis/dropify/blob/master/src/js/dropify.js#L166
    - Línea 99: https://github.com/JeremyFagis/dropify/blob/master/src/js/dropify.js#L99


## JavaScript `File` y `FileReader`
- File docs: https://developer.mozilla.org/en-US/docs/Web/API/File
- FileReader docs: https://developer.mozilla.org/en-US/docs/Web/API/FileReader
- Mostrar miniaturas de imágenes: https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications#Example_Showing_thumbnails_of_user-selected_images



## Cómo organizar código HTML dentro de un string JavaScript
- Una muy buena forma es usar separador multi linea: https://stackoverflow.com/a/16270826/1407371


## Cómo leer los atributos *data* de un elemento con vanilla JS
- SO: https://stackoverflow.com/questions/33760520/get-data-attributes-in-javascript-code#33760558
- Usar `el.dataset.attr_name`: `el.dataset.id`
- Usando `el.getAttribute(attr_name)`


## ¿Cómo reemplazaar el `trigger` de jQuery en vanilla JS?

Se puede con `dispatchEvent()` pero no doy para hacerlo funcar en algo que quiero probar.
Enlaces:

- SO: https://stackoverflow.com/questions/3368578/trigger-a-keypress-keydown-keyup-event-in-js-jquery
- dispatchEvent docs: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent


# Error de ES6 no habilitado en `precompile` de assets

El error:

    Uglifier::Error: Unexpected token: operator (>). To use ES6 syntax, harmony mode must be enabled with Uglifier.new(:harmony => true)

Solución:
Dada en GitHub: https://github.com/lautis/uglifier/issues/127#issuecomment-352224986

    config/environments/production.rb
    
    # cambiar
    config.assets.js_compressor = :uglifier
    
    # por
    
    config.assets.js_compressor = Uglifier.new(harmony: true)

