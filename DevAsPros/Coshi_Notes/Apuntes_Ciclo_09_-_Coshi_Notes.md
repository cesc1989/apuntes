# Apuntes Ciclo 09 - Coshi Notes

# Sobre las media queries

Solo dejaré enlaces:

- [A Complete Guide to CSS Media Queries](https://css-tricks.com/a-complete-guide-to-css-media-queries/)
-  [Media Queries for Standard Devices](https://css-tricks.com/snippets/css/media-queries-for-standard-devices/) 


# Usar explorador de archivo para cargar imagen al editor Tiptap

Aquí la clave era entender cómo usar el campo tipo file para montar la imagen. Al final, sirvió parte de lo que ya había hecho y había entendido.

El resumen es más o menos:

- Mostrar una barra de herramientas dentro del editor
- Uno de los botones tiene un input file oculto, cuando se clica se abre el explorador de archivos
- Al elegir un archivo este se sube al servidor y capturamos la URL devuelta
- La URL se pega en el editor mediante al extensión Image

Al final quedé con dos extensiones para captura de imágenes:

- CustomImage
- Image

El primero es para soportar pegar y arrastra. El segundo para la carga desde el explorador.

De este [gist](https://gist.github.com/fdrissi/ef51cc8cf995115148923d1ee12f72fe) saqué prácticamente toda la base para lo que hice.


## La Implementación

Agregué tres funciones adicionales al controlador `rich-editor`:

```javascript
  fileFromExplorer(event) {
      if (!event?.target?.files?.[0]) return
    
      this.uploadFile(event.target.files[0])
        .then(res => res.json())
        .then(res => this.addImageToEditor(res['result']))
        .catch(err => console.error(err))
    }
    
    addImageToEditor(imageUrl) {
      if (imageUrl) {
        console.log("Entramos con ", imageUrl)
        // this.element.editor.commands.setImage({ src: imageUrl })
        this.element.editor.chain().focus().setImage({ src: imageUrl }).run()
      }
    }
    
    launchFileExplorer(event) {
      event.preventDefault()
    
      this.fileExplorerTarget.click()
    }
```

La función  `fileFromExplorer` se encarga de capturar la imagen seleccionada.

La imagen está en la lista `files` de `event.target`:

```javascript
event.target.files[0]
```


> Nota como se reusa `this.uploadFile` la cual es la función que se hizo para el módulo `dropImage`.

La otra función es `addImageToEditor(imageUrl)`. Esta es la que, cuando hay una URL, la pega en el editor para que se muestre una etiqueta <img />.

La clave es tener acceso al editor:

```javascript
this.element.editor.chain().focus().setImage({ src: imageUrl }).run()
```

**Un poco de detalles:**

1. la función `chain()` permite que las demás se encadenen. [Docs](https://tiptap.dev/docs/editor/api/commands#chain-commands).
2. La función `focus()` hace que el editor recupere el foco luego de perderlo al abrir el explorador de archivos
3. La función `setImage()` viene del módulo Image, hace que se agregue la etiqueta <img src=src /> para poder mostrar una imagen
4. Finalmente, `run()` hace que se ejecute toda la cadena de funciones.

De último pero no menos importante es la función `launchFileExplorer(event)`. Esta es ejecutada por el botón que tiene el ícono de adjunto. Se abre el explorador mediante un input file oculto:

```html
# El botón
<button class="btn btn-outline-secondary" data-action="rich-editor#launchFileExplorer" title="Adjuntar imagen">
	<i class="fas fa-paperclip"></i>
</button>

# El campo oculto
<input
	type="file"
	id="file_explorer"
	class="d-none"
	accept="image/png, image/jpeg"
	data-rich-editor-target="fileExplorer"
	data-action="rich-editor#fileFromExplorer"
/>
```
