# Apuntes Ciclo 22 - Truncar texto con Rails y CSS

## Truncando con Rails o CSS

Para las vistas de Favorites o Todos el contenido del comentario era truncado con Rails así:
```ruby
<%= comment.plain_content.truncate(400).html_safe %>
```

Eso funciona muy bien. Recortaba la cantidad de texto para comentarios muy largos y también mantenía el HTML para que se viera bonito sin ir al comentario.

Sin embargo, esto tenía un problema y es que en algunos comentarios podría haber un mal escapado si el recorte se hacía dentro de una etiqueta HTML sin cerrar. Ejemplo:
```html
<div class="card-body comment-card"> <p class="card-text"> </p><p><s>Se dañó el disco duro. Voy a comprar uno SSD y las películas y otras cosas pesadas las mant... </s></p><s> </s></div>
```

Esto lo puedo solucionar de dos formas: con Rails o CSS.

### Forma Rails

Seguimos usando truncate pero de otra forma:
```ruby
<%= sanitize(truncate(comment.plain_content, length: 400, escape: false)) %>
```

Con el `escape: false` se evita recortar el HTML e igual se sigue truncando el texto.

### Forma CSS

Esta me gustó por sencilla y logra lo mismo:
```css
.text-truncate-multiline {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
  max-height: 4.5em;
  line-height: 1.5em;
}
```

#### ¿Qué son esos estilos -webkit-x?

Según DeepSeek.

##### 1. `display: -webkit-box;`

- **Qué hace**: Activa el "CSS Box Model" antiguo (Flexbox anterior)
- **Por qué**: Necesario para que funcionen `-webkit-line-clamp` y `-webkit-box-orient`
- **Compatibilidad**: Funciona en todos los navegadores modernos a pesar del prefijo `-webkit-`

##### 2. `-webkit-line-clamp: 3;`

- **Qué hace**: Limita el texto a máximo 3 líneas
- **Detalle**: Si el texto ocupa más de 3 líneas, se corta y muestra "..."
- **Flexibilidad**: Puedes cambiar `3` por cualquier número (ej: `2`, `4`, etc.)

##### 3. `-webkit-box-orient: vertical;`

- **Qué hace**: Organiza el contenido en dirección vertical (de arriba a abajo)
- **Importante**: Esta propiedad es necesaria para que `line-clamp` funcione

