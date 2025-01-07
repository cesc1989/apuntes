# RFC 003: Comentario como Tarea

He escrito muchos comentarios con la forma “Pendiente” + checklist. Creo que esto evidencia un patrón claro y una necesidad que puedo resolver para mejorar mi uso de Coshi Notes.

Si le doy una característica extra a los comentarios para que funcionen al estilo "To Do", tendré los comentarios donde puedo poner todo tipo de detalles y combinarlos con "todos" para saber qué tengo pendiente.

Ejemplos de comentarios pendientes ahora mismo:

- Lista de películas de Werner Herzog por ver
- Lista de películas para descargar para la USB
- Lista de anime por descargar

## Detalles e Ideas

Si hago esta funcionalidad, puedo hacer comentarios de tipo "todo" que destaquen de los demás. Similar a como un comentario "Favorito" destaca.

Un comentario tipo “Pendiente” tendrá un color diferencial cuando esté _pendiente_ y cuando esté completo.

Un comentario tipo “Pendiente” se podrá completar.

Habría un menú de navegación en el sidebar para ir a ver todos los comentarios de Tipo "Todo"

## Posible Implementación

Lo veo de esta forma.

Creo un enum de estado para el comentario.

```ruby
enum state: {
  todo: 2,
  done: 3
}
```

Con los estados `todo` y `done` puedo jugar.

Puedo listar en una página aparte todo comentario que sea `todo`. En la página de Tema, el comentario `todo` tendrá un estilo diferenciador. Cuando estén `done` también lo tendrá.

Para convertir a `todo` y marcar como `done` podrá haber botones. Para convertir a `todo` habrá un botón en el dropdown de opciones del comentario.

Un comentario en modo `todo` tendrá un botón para marcar como `done`.

El clicar el botón para marcar como `done` cambiara de color. El botón podrá ser estilo toggle para no tener que lidiar con un modal de confirmación.