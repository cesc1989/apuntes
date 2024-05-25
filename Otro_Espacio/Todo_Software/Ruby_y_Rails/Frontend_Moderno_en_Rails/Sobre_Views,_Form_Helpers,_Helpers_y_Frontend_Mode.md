# Sobre Views, Form Helpers, Helpers y Frontend Moderno en Rails
La idea de este documento es tener contenido con respecto a vistas, pruebas para vistas (sea unit o e2e), aclarar como funcionan los helpers y sobre todo los form helpers y, finalmente, todo lo que sea frontend estrictamente en Rails: Hotwire, Stimulus, Turbolinks, Webpacker, etc.

# Cómo usar el helper `collection_select`

En la [documentación](https://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_select) lo define así:

    collection_select(object, method, collection, value_method, text_method, options = {}, html_options = {})

Este [artículo desglosa](https://theresamorelli.medium.com/collection-select-what-the-heck-4e1cabc4be4b) las diferentes opciones. Veamos:

    <%= f.collection_select(:category_id, Category.all, :id, :name, { include_blank: 'Escoge' }, { prompt: 'Elige', selected: @activity.category_id }) %>

**object** -> corresponde al modelo instanciado para el formulario. En el caso de `form_with` se puede omitir el parámetro.

**method** → configura el nombre e ID del campo. En el ejemplo `category_id`.

**collection** → define el listado de *option choices* de la lista de selección.

**value_method** → es el atributo `value` en el `<option>`.

**text_method** → es el atributo de texto para el `<option>`.

**options = {}** → la documentación específica los símbolos:

    :include_blank
    :prompt
    :index -> se usa en los html_options.
    :disabled

**html_options = {}** →

> Al respeto de `prompt` o `include_blank`. [+Lecciones y Aprendizajes #2: ¿Cuál-es-la-diferencia-entre-i](https://paper.dropbox.com/doc/Lecciones-y-Aprendizajes-2-Cual-es-la-diferencia-entre-i-fgGrUwoQeOUTHyeas6zSP#:uid=319428766128849982254855&amp;h2=%C2%BFCu%C3%A1l-es-la-diferencia-entre-i) 

