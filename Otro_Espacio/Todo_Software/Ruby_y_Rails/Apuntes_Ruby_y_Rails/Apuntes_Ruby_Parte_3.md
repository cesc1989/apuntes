# Apuntes Ruby. Parte 3

## Obtener horas a partir de segundos
- [Stack Overflow](https://stackoverflow.com/questions/3963930/ruby-rails-how-to-convert-seconds-to-time)
- [Gist](https://gist.github.com/shunchu/3175001)


    Time.at(seconds).utc.strftime("%H:%M:%S")

Y cómo realmente debe ser: https://gist.github.com/shunchu/3175001#gistcomment-1465351

Ver también: [milisegundos a horas](https://stackoverflow.com/questions/40485828/convert-milliseconds-to-hours-and-mins-in-ruby/40486133#40486133)

## Enumerable first or find returns array when send to a hash object

¿Por qué `Enumerable#first` devuelve un array al aplicarlo sobre un Hash?

- [Stack Overflow](https://stackoverflow.com/questions/16250746/why-does-enumerablefind-detect-return-an-array-even-when-called-on-an-hash)

Probado en [Replit](https://repl.it/@cesc89/Enumerable-returns-array-when-applied-on-hash).

## No iterar un array e ir borrando sus elementos al mismo tiempo

¿Por qué?

    my_arr = *1..10
    my_arr_copy = my_arr.clone
    
    # When the element in index 0 is deleted, the index of the whole array is changed. At first, 1 is index 0, but when it gets deleted from the array, now number 2 belongs to index 0 and becomes the "actual" element being iterated.
    #
    # So, when the second iteration starts, what should be element in index 1 (number 2) is now element in initial index, number 3.
    my_arr.each_with_index do |i, ind|
      puts "Ele: #{i} - Index: #{ind}"
      my_arr.delete_at(0)
      puts "Ele: #{i} - Index: #{ind}"
    end
    
    # Here, by using the actual index of the element, the missbehavior is corrected
    my_arr_copy.each_with_index do |i, ind|
      puts "A - Ele: #{i} - Index: #{ind}"
      my_arr.delete_at(ind)
      puts "D - Ele: #{i} - Index: #{ind}"
    end

Como se menciona en [esta respuesta](https://stackoverflow.com/a/21214030/1407371) en Stack Overflow:

> Do not change underlying container while you iterate it. This could cause unexpected behaviour.

Enlaces:

- [Un replit al respecto](https://repl.it/@cesc89/Arraydeleteatindex)
- [Stack Overflow 1](https://stackoverflow.com/questions/35499378/delete-element-of-array-while-iterating)


## `String#to_i` devuelve 0 si comienza con una letra

Ver [String](https://apidock.com/ruby/String/to_i)

> Returns the result of interpreting leading characters in str as an integer base base (between 2 and 36). Extraneous characters past the end of a valid number are ignored. If there is not a valid number at the start of str, 0 is returned. This method never raises an exception when base is valid.

Esto puede verse como una forma de saber si un string tiene solo números.

También en [Stack Overflow](https://stackoverflow.com/questions/8768865/why-does-rubys-stringto-i-sometimes-return-0-when-the-string-contains-a-number).

## Array#all?

Este método solo con leer de la impresión de que [valida de que en un array haya elementos válidos](https://ruby-doc.org/core-2.5.3/Enumerable.html#method-i-all-3F). Sin embargo, en la documentación explica que cuando el array está vacío, el método recibe un objeto `obj` que evalúa como verdadero y por ende `[].all?` retorna verdadero.

> The method returns `true` if the block never returns `false` or `nil`. If the block is not given, Ruby adds an implicit block of `{ |obj| obj }` which will cause [all?](https://ruby-doc.org/core-2.5.3/Enumerable.html#method-i-all-3F) to return `true` when none of the collection members are `false` or `nil`.


## Así se escriben macros en Ruby

[Gist](https://gist.github.com/cesc1989/f91be1fa7d6091bede0426753ba8eb99).

    module SaysHi
      def self.included(base)
        base.extend ClassMethods
      end
    
      module ClassMethods
        def hi_to
          @hi_to ||= ""
        end
    
        def says_hi_to(name)
          @hi_to = name
        end
      end
    
      def say_hi
        puts "Hello, #{self.class.hi_to}"
      end
    end
    
    class HelloReader
      include SaysHi
      says_hi_to "dear reader"
    end
    
    class HelloWorld
      include SaysHi
      says_hi_to "world"
    end
    
    HelloReader.new.say_hi # => "Hello, dear reader"
    HelloWorld.new.say_hi # => "Hello, world"


## ¿Cómo saber que una cadena es solo números?

Visto en [Mentalized](https://mentalized.net/journal/2011/04/14/ruby-how-to-check-if-a-string-is-numeric/).

Pensaba hacerlo con una expresión regular pero así pasaba en Rubular:

![](https://paper-attachments.dropbox.com/s_CBD537F71EE9B7395514CCEB345BF327B497A21AE9C257B3A25F75143E28AC43_1618328486567_image.png)


El post sugiere hacerlo así:

    class String
      def numeric?
        Float(self) != nil rescue false
      end
    end

Y en [Rosetta Code](https://rosettacode.org/wiki/Determine_if_a_string_is_numeric#Ruby) así:

    def is_numeric?(s)
        !!Float(s) rescue false
    end

Llamativo que el método `Float` es del módulo [Kernel](https://ruby-doc.org/core-3.0.0/Kernel.html#method-i-Float).

## ¿Tiene Ruby primitivas?

[Stack Overflow](https://stackoverflow.com/questions/18790442/are-there-primitive-types-in-ruby). Respuesta: No.

> There are no primitive data types in Ruby. Every value is an object, even literals are turned into objects


        nil.class  #=> NilClass
       true.class  #=> TrueClass
      'foo'.class  #=> String
       :bar.class  #=> Symbol
        100.class  #=> Integer
       0x1a.class  #=> Integer
    0b11010.class  #=> Integer
      123.4.class  #=> Float
    1.234e2.class  #=> Float
## Difference between `require` and `require_relative`

[Stack Overflow](https://stackoverflow.com/questions/3672586/what-is-the-difference-between-require-relative-and-require-in-ruby). Y de [esta respuesta](https://stackoverflow.com/a/29865560/1407371).


- Use `require` for installed gems
- Use `require_relative` for local files

`require` uses your `**$LOAD_PATH**` to find the files.
`require_relative` uses the current location of the file using the statement.

**require**
Require relies on you having installed (e.g. `gem install [package]`) a package somewhere on your system for that functionality.

When using `require` you *can* use the "`./`" format for a file in the current directory, e.g. `require "./my_file"` but that is not a common or recommended practice and you should use `require_relative` instead.

**require_relative**
This simply means include the file 'relative to the location of the file with the require_relative statement'. I *generally* recommend that files should be "within" the current directory tree as opposed to "up", e.g. **don't** use:


    require_relative '../../../filename'
## Instance methods: why can I use getter without `self`, but setter only with `self`

[Stack Overflow](https://stackoverflow.com/questions/19617169/ruby-instance-methods-why-can-i-use-getter-without-self-but-setter-only-with).

Se refiere a esto:

    def getter
      name
    end
    
    def setter
      self.name = "nuevo"
    end

Sino se usa `self` al invocar el método name, el interpreta creerá que se está definiendo una varibale local.

## ¿Es `0` falso o verdadero?

[Stack Overflow](https://stackoverflow.com/questions/16495697/0-is-false-in-rails-why). Respuesta: verdadero.

> In ruby there are only two values that evaluate to false in logical expressions: false and nil. Since 0 is neither of them, it evaluates to true and thus !true equals false.
## Error de Instalación de Puma

Sí, falló la instalación de Puma! El error:

    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
    
    An error occurred while installing puma (4.3.1), and Bundler cannot continue.
    Make sure that `gem install puma -v '4.3.1' --source 'https://rubygems.org/'` succeeds before bundling.
    
    In Gemfile:
      puma

La solución está dada por:

    gem install puma:4.3.1 -- --with-cflags="-Wno-error=implicit-function-declaration"

Tomado de [este issue](https://github.com/puma/puma/issues/2304#issuecomment-664448309).

Había encontrado en [Stack Overflow esto](https://stackoverflow.com/questions/35329451/cant-install-puma-gem-on-osx-10-11-3) que sugiere que se instalara Brew y se enlazara pero no alcanzamos a probarlo.

Este error fue arreglado desde [la versión 4.3.6](https://github.com/puma/puma/issues/2342#issuecomment-709322223).

## Cómo abrir una gema instalada localmente para inspección?

Digamos que quiere ver el código fuente de una gema, ¿cómo hago para abrirla desde la línea de comandos?

Según [Stack Overflow](https://stackoverflow.com/questions/21145216/how-can-i-see-the-source-code-of-a-gem-installed-on-my-machine) puedo hacer así:

    $ bundle open devise

pero me pide que configure la variable $EDITOR

    To open a bundled gem, set $EDITOR or $BUNDLER_EDITOR

Se hace así:

    $ EDITOR=subl bundle open devise

Otra forma es primero encontrando la ruta absoluta de la gema y luego usar el editor para abrir dicha carpeta:

    $ bundle show devise
    /Users/fquintero/.rvm/gems/ruby-2.7.1/gems/devise-4.8.1

y luego:

    $ subl /Users/fquintero/.rvm/gems/ruby-2.7.1/gems/devise-4.8.1

