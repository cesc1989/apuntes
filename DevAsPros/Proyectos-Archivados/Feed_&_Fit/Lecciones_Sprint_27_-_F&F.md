# Lecciones Sprint 27 - F&F

## Ejecutar bundle indicando la versión

Cómo ejecutar `bundle` [con una versión en específica](https://makandracards.com/makandra/9741-run-specific-version-of-bundler) y sin tener que desinstalar una más nueva.


    $ bundle _1.17.3_ install


## Agregar atributos data a elementos *option* de una lista de selección

Al usar el helper `select` se pasa un *array* de opciones. En ese *array* [si hay un hash, ese se convierte a atributos HTML del](https://stackoverflow.com/questions/7624824/rails-formtastic-adding-data-field-to-option-tag) [*option*](https://stackoverflow.com/questions/7624824/rails-formtastic-adding-data-field-to-option-tag)*.*


    def product_options_tags(products)
      products.map do |product|
        [
          product.name,
          product.id,
          data: {
            size_selected: product.size_selected,
            target_selected: product.target_selected
          }
        ]
      end
    end

También descubrí [el helper](https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html#method-i-tag) `[tag](https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html#method-i-tag)` pero generaba *options* vacios por lo que no lo usé.


## Obtener opción elegida en una lista de selección con jQuery

Esto es [diferente a obtener el valor elegido](https://stackoverflow.com/questions/10659097/jquery-get-selected-option-from-dropdown) con `$lista.val()` ya que podría querer el texto de la opción o un atributo data:


    $('select').find(":selected").text();
    
    $('select').find(":selected").data('attribute');

