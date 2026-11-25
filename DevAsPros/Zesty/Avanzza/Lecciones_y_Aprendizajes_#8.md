# Lecciones y Aprendizajes #8

- Función `jQuery(``'``$``'``)` con selector de contexto https://api.jquery.com/jQuery/#jQuery
- Sobre factories para `has_one` [https://qiita.com/Kolosek/items/7c85337f928161e0e76e](https://qiita.com/Kolosek/items/7c85337f928161e0e76e)
- Configuración de `strftime` cuando se usa el helper de I18N de Rails
    - https://guides.rubyonrails.org/i18n.html#adding-date-time-formats
- Playing with Ruby Dates https://robdodson.me/playing-with-ruby-dates/
## `content_tag` helper para crear etiquetas HTML
- ActionView::Helpers::TagHelper https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html


## Cómo transformar el texto de una opción de un select

Cuando se indica el `label_method` o el `text_method` de un collection_select, este se espera que sea un símbolo que identifique el nombre del método de la instancia.

Si se llama `upcase` en el simbolo, el objeto no identifica un método todo en mayúsculas. No es lo mismo `#NAME` que `#name`.

La solución https://stackoverflow.com/questions/4228382/using-capitalize-on-a-collection-select


## Cómo asignar un valor por defecto a un campo sin hacerlo por base de datos
- Una forma es en el controlador antes de que se guarde
    - pero es feo
- Otra forma es con el callback de modelo `[after_initialize](https://stackoverflow.com/a/5127684/1407371)`
    - también es feo
- La otra es con el macro `[attribute](https://stackoverflow.com/a/43484863/1407371)` [en el modelo que está desde Rails 5+](https://stackoverflow.com/a/43484863/1407371)
    - `ActiveRecord#attribute` https://api.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html#method-i-attribute


## Rango de fechas

Al crear rango de fechas hay que asegurarse de llamar el método `#to_date` en cada extremo del rango para evitar la excepción:

    TypeError: can't iterate from ActiveSupport::TimeWithZone

Visto en [Stack Overflow](https://stackoverflow.com/questions/21744683/ror-cant-iterate-from-datetime-timewithzone)


## Self join con condiciones
- http://danielchangnyc.github.io/blog/2013/11/06/self-referential-associations/
- https://edgeguides.rubyonrails.org/association_basics.html#scopes-for-has-many
- https://stackoverflow.com/questions/20307874/what-is-the-equivalent-of-the-has-many-conditions-option-in-rails-4



## Strategy Design Pattern

Patrón de diseño aplicable:

- [Strategy Pattern](https://github.com/kamranahmedse/design-patterns-for-humans#-strategy)
- [Design Patterns in Ruby](https://github.com/davidgf/design-patterns-in-ruby/blob/master/strategy.md)



# Sobre métodos en namespace

Resulta que los métodos definidos en un módulo de un namespace, no son visibles en las clases de dicho namespace sino se incluyen mediante *include, prepend o extend*
Esto es porque: [Ver respuesta](https://stackoverflow.com/a/14501884/1407371)

> Namespaces in Ruby don’t work the way you seem to think.
> 
> There is no such thing as a “namespace method”. A::B#foo is an instance method on the module A::B—which is a module named B in the namespace of A.
> 
> Namespaces modules/classes have no special relationship of inheritance between them. They are purely organizational in nature, except when defined the long way (e.g. module A; module B; end; end) when they can affect lexical scope.
> 
> If you want to get methods of A::B in A::B::C, you must include A::B in A::B::C, just like you would anywhere else. You have to do this because, as said above, there's nothing special about a namespaced module/class, it is treated the same as any other.



## Cómo listar métodos en un módulo sin listar los heredados por defecto?

Con `#public_instance_methods` o `#private_instance_methods`
Ver https://stackoverflow.com/questions/3362143/list-methods-only-in-a-module


## Cómo sobreescribir un método dado por un atributo de ActiveRecord

En [Stack Overflow](https://stackoverflow.com/questions/373731/override-activerecord-attribute-methods) se mencionan varias alternativas.

Así:

    def plate
      self[:plate].upcase
    end

o así:

    def name=(name)
      write_attribute(:name, name.capitalize)
    end
    
    def name
      read_attribute(:name).downcase  # No test for nil?
    end

