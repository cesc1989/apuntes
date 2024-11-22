# RFC 002: Búsqueda Mejorada

La búsqueda y los resultados están muy enfocados en el Tema. Si bien está configurada la búsqueda para encontrar cosas en Tema y Comentario, todo termina siendo más prominente para lo primero. Quiero cambiar eso para que el Comentario tenga más prominencia. Al fin y al cabo, la mayor cantidad de detalles están en los comentarios.

Así luce el controlador de búsqueda:
```ruby
class SearchController < ApplicationController
  def index
    @q = Theme.ransack(params[:q])
    @q.sorts = 'created_at desc' if @q.sorts.empty?

    @themes = if params[:q].present?
                @q.result(distinct: true).includes(:channel, :comments)
              else
                @themes = []
              end
  end
end
```

El setup de Ransack permite buscar en Tema y Comentario:
```ruby
# Tema
def self.ransackable_attributes(auth_object = nil)
	%w[
		name
		description
		closed
		created_at
		updated_at
	]
end

def self.ransackable_associations(auth_object = nil)
	%w[
		comments
	]
end

# Commentario
def self.ransackable_attributes(auth_object = nil)
    %w[
        plain_content
        created_at
      ]
end
```

Creo que la búsqueda debería ser al revés. Donde el centro de esta es el modelo Comment:
```ruby
@q = Comment.ransack(params[:q])
```

Sin embargo, hay veces que el título o descripción del tema tiene datos clave para encontrar lo que busco. Así que no los puedo descartar.

## Modelo Search: Active Record o PORO

La solución a esto es, como he visto en algunas lecturas, crear un modelo que sea el que reciba la búsqueda. Este modelo guardará textos del título y descripción del tema, también el texto del comentario.

Sería un modelo polimorfíco:
```ruby
Search # tabla searches
title: string # Título del Tema
body: text # Descripción del Tema o Texto del Comentario
searcheable_id # el ID de la relación: Tema o Comentario
searchable_type: # la clase de la relación: Tema o Comentario
```

En callbacks, se configura para que se guarden el contenido en los campos correspondientes y así la búsqueda se hace sobre el modelo `Search` en lugar de `Theme` o `Comment`.

Así es el schema resultante del [ejemplo](https://github.com/stevepolitodesign/rails-search-across-multiple-models/blob/main/db/schema.rb#L22-L30) de Steve Polito:
```ruby
  create_table "search_entries", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.string "searchable_type", null: false
    t.integer "searchable_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["searchable_type", "searchable_id"], name: "index_search_entries_on_searchable"
  end
```


## Enlaces

Este se acerca más a lo que quiero:
- [Search Across Multiple Models in Rails](https://stevepolito.design/blog/search-across-multiple-models-in-rails)

Estos dan ideas y soporte a cómo montar la búqueda:

- [Implementing Multi-Table Full Text Search with Postgres in Rails](https://thoughtbot.com/blog/implementing-multi-table-full-text-search-with-postgres): usa Postgres
- [Complex Object Searching in Rails with Ransack](https://lewiseason.co.uk/2017/05/12/complex-object-searching-rails)
- [Search and Filter Rails Models Without Bloating Your Controller](https://www.justinweiss.com/articles/search-and-filter-rails-models-without-bloating-your-controller/)