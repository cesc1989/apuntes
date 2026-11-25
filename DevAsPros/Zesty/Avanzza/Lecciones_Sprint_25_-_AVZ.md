# Lecciones Sprint 25 - AVZ

## Sobre event.relatedTarget

Para pasar información, de manera más sencilla, de la fila en la tabla de conductores al modal(sin tener que hacer un modal por cada registro como hice en Feed & Fit), podemos probar esta forma:

```javascript
//triggered when modal is about to be shown
$('#my_modal').on('show.bs.modal', function(e) {
  //get data-id attribute of the clicked element
  var bookId = $(e.relatedTarget).data('book-id');

 //populate the textbox
  $(e.currentTarget).find('input[name="bookId"]').val(bookId);
});
```

Vista en [Stack Overflow](https://stackoverflow.com/a/25060114/1407371)

**Más info al respecto**

- Otra pregunta en [Stack Overflow sobre un ejemplo de uso de](https://stackoverflow.com/questions/45731337/pass-data-attribute-to-modal-bootstrap) `[event.relatedTarget](https://stackoverflow.com/questions/45731337/pass-data-attribute-to-modal-bootstrap)`
- Documentación de `event.relatedTarget` [en MDN](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/relatedTarget)
- En la [documentación del Modal en Bootstrap](https://getbootstrap.com/docs/4.3/components/modal/#events) explica que en el evento `show.bs.modal`, se tiene acceso a al elemento que lanza el modal mediante un click a través de `event.relatedTarget`

## Sobre el objeto Flash en las peticiones en Rails

El hash Flash sirve para pasar información tipo texto entre una petición y otra.

**Artículos**
- Sobre el [Flash hash](http://stevenleiva.com/flash-messages-in-rails) de las peticiones en Rails
- Cómo [salirse temprano de una acción](https://blog.arkency.com/2014/07/4-ways-to-early-return-from-a-rails-controller/) de controlador


## Variados
- Encontrar [input de un tipo dado y con una clase determinada](https://stackoverflow.com/questions/1065925/jquery-selecting-by-class-and-input-type) usando jQuery

