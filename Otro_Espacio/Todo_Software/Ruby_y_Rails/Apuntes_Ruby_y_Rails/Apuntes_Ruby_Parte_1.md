# Apuntes Ruby. Parte 1

## Sobre la diferencia entre los métodos de String `tr` vs `gsub`

Según esta respuesta en [Stack Overflow](https://stackoverflow.com/questions/26749065/what-is-the-difference-between-tr-and-gsub#26750460).

Sobre `#tr`:

> Use `[tr](http://ruby-doc.org/core/String.html#method-i-tr)` when you want to replace (translate) single characters.

Esto ocurre porque el método `#tr` empareja de a un caracter (sin usar expresiones regulares), por esto, los caracteres no necesitan estar en el mismo orden del primer párametro.

    'abcde'.tr('bda', '123')
    #=> "31c2e"
    
    'abcde'.tr('bcd', '123')
    #=> "a123e"

Y sobre gsub:

> Use `[gsub](http://ruby-doc.org/core/String.html#method-i-gsub)` when you need to use a regular expression or when you want to replace longer substrings.

Ejemplo:

    'abcde'.gsub(/bda/, '123')
    #=> "abcde"
    
    'abcde'.gsub(/b.d/, '123')
    #=> "a123e"

Finalmente, en [este comentario](https://stackoverflow.com/questions/26749065/what-is-the-difference-between-tr-and-gsub#comment59242866_26750460) explican que `#tr` significa “translate”.

## Diferencia entre módulo y clase

En la respuesta elegida dan esta definición:

> **Modules are about providing methods** that you can use across multiple classes - think about them as "libraries" (as you would see in a Rails app). **Classes are about objects; modules are about functions.**

Se puede complementar con la tabla de [esta otra](https://stackoverflow.com/a/9778021/1407371) respuesta.


## Cómo inicializar una clase en un bloque

El [OP](https://stackoverflow.com/questions/9826208/how-to-initialize-class-in-block-in-ruby) intenta inicializar una clase dentro de un bloque de esta forma:

    class Foo 
      attr_accessor :bar
    end
    
    obj = Foo.new do |a|
      a.bar = "baz"
    end
    
    puts obj.bar

La forma correcta es usando el método `#tap`:

    class Foo 
      attr_accessor :bar
    end
    
    obj = Foo.new.tap do |a|
      a.bar = "baz"
    end
    
    puts obj.bar

Lo que nos lleva a ver

## Método `#tap`.

Según este [artículo en Medium](https://medium.com/aviabird/ruby-tap-that-method-90c8a801fd6a):

> It allows you to do something with an object inside a block, and always have the block return the object itself.
> 
> It was created for tapping into method chains.

Y en palabras de [Leigh Halliday](https://www.leighhalliday.com/implementing-rubys-tap-method):

> allows you to "tap into" a method chain, modify an object and receive that same object as the result.


## Variables de clase y variables de instancia

[Stack Overflow](https://stackoverflow.com/questions/3802540/difference-between-class-variables-and-class-instance-variables).

> A class variable (`@@`) is shared among the class and all of its descendants. A class instance variable (`@`) is not shared by the class's descendants.

[Stack Overflow](https://stackoverflow.com/questions/9311347/using-instance-variables-in-class-methods-ruby#9311665). Sobre clase con variables de instancia.

> The reason instance variables work on classes in Ruby is that Ruby classes are instances themselves.
## Integer to binary

[Stack Overflow](https://stackoverflow.com/questions/2339695/how-to-convert-a-string-or-integer-to-binary-in-ruby#2340415).

    9.to_s(2) #=> "1001"
    
    # "%b" % number
    "%b" % 245
    => "11110101"
## Variables globales en ruby

[Stack Overflow](https://stackoverflow.com/questions/1042384/how-do-you-use-global-variables-or-constant-values-in-ruby).

De esta [respuesta](https://stackoverflow.com/a/1047482/1407371):

> Variable scope in Ruby is controlled by sigils to some degree. Variables starting with `$` are global, variables with `@` are instance variables, `@@` means class variables, and names starting with a capital letter are constants. All other variables are locals. When you open a class or method, that's a new scope, and locals available in the previous scope aren't available.


## Sigils in Ruby

[Mikey Hogarth](https://mikeyhogarth.wordpress.com/2011/11/07/ruby-sigils/)

> acually had no idea that non-alphanumeric characters decorating variables were known as sigils in computer science, but sure enough they are!
> 
> That is, sigils that accompany variables. These will always go before a variable, never after. Ruby is generally more forgiving than other languages in its naming conventions for things like methods and variables

[Wikipedia](https://en.wikipedia.org/wiki/Sigil_(computer_programming))

> is a symbol affixed to a variable name, showing the variable's datatype or scope, usually a prefix, as in $foo, where $ is the sigil.

Variable sigils:

    @name, @age
    $settings
    @@last_invoked
    *args
    &block

Method sigils:

    even?
    sort!


## Clases dentro de Clases en Ruby

He visto en muchas librerías que hacen cosas como:

    class Exterior
      def prueba_exterior
        puts "Desde afuera"
      end
    
      class Interior
        def prueba_interior
          puts "desde adentro"
        end
      end
    end

Según esta [respuesta](https://stackoverflow.com/a/14740440/1407371) y este [comentario](https://stackoverflow.com/questions/14739640/ruby-classes-within-classes-or-modules-within-modules#comment20626988_14740440), en realidad no hay ningún tipo de anidamiento ni tampoco herencia.

El interprete va apilando una definición dentro del namespace de la anterior. Así:

    Exterior = Class.new do 
      def prueba_exterior
        puts "Desde afuera"
      end
    
      Interior = Class.new do   
        def prueba_interior
          puts "desde adentro"
        end
      end
    end

Cuando se invoca `Exterior::Interior.new` solamente se está haciendo un llamado a la clase `Interior` que está dentro del namespace de la clase `Exterior`.

    p Exterior::Interior.ancestors
    
    [Exterior::Interior, Object, Kernel, BasicObject]

Solo sería una forma de organizar clases.

Recursos:

- [Ruby Classes within Classes (or Modules within Modules)](https://stackoverflow.com/questions/14739640/ruby-classes-within-classes-or-modules-within-modules)
- [When to use nested classes and classes nested in modules?](https://stackoverflow.com/questions/6195661/when-to-use-nested-classes-and-classes-nested-in-modules)


## Hacer una clase anidada que sea privada

Quisiera que la clase más interna sea privada, solo invocable desde la clase que la contiene:

    class Exterior
      class Interior
      end
    
      private
    
      class SuperInterior
        def muy_adentro
          puts "muuuy adentro"
        end
      end
    end

Si hago esto, no hay en realidad una clase interna privada:

    muy_adentro = Exterior::SuperInterior.new
    p muy_adentro.muy_adentro
    
    muuuy adentro
    nil

Para impedir ese comportamiento hay que usar `private_constant`

    class SuperInterior
      def muy_adentro
          puts "muuuy adentro"
      end
    end
    private_constant :SuperInterior
    
    # Ejecución
    
    muy_adentro = Exterior::SuperInterior.new
    p muy_adentro.muy_adentro
    
    Traceback (most recent call last):
    main.rb:38:in `<main>': private constant Exterior::SuperInterior referenced (NameError)

En todo caso no es garantía de que no se pueda usar del todo.

Recursos:

- [How to implement private inner class in Ruby](https://stackoverflow.com/a/39716839/1407371)


## Include, Prepend y Extend de módulos

[SitePoint](https://www.sitepoint.com/get-the-low-down-on-ruby-modules/). La diferencia entre `include`, `prepend` y `extend` de módulos.

**include**
Cuando se hace `include` a un módulo en una clase, los métodos de ese módulo se agregan a la super clase de la clase, es decir, una instancia de la clase X puede recibir el llamado de los métodos *incluidos* del módulo pero solo si la clase X no los tiene definidos.

**prepend**
La principal diferencia entre `include` y `prepend` es que este último puede "sobre escribir" los métodos definidos en la clase. Si la clase X tiene definido el método `letra` y el módulo `Consonante` tiene un método de mismo nombre, al hacer:

    Class X
      prepend Consonante
    
      def letra
        "del modulo"
      end
    end

Una instancia de X recibiendo el método `letra` llamará al definido en el módulo y no en la clase.

**extend**
Al extender una clase, es decir, agregar los métodos del módulo usando la instrucción `extend`, las instancias de la clase no pueden recibir dichos métodos pero sí la clase. `extend`, por ende, extiende a la clase permitiéndole recibir métodos adicionales.

> Es posible extender una instancia usando el método `extend`
> `my_instance.extend(ModuleName).method_call`


## Ruby Garbage Collector

From this error: `[BUG] object allocation during garbage collection phase`

- [Not ready for production](http://500errors.com/omniref/2014-03-27-ruby-garbage-collection-still-not-ready-for-production.html) and in [Hacker News](https://news.ycombinator.com/item?id=7488233)


## Cuándo usar `Struct` u `OpenStruct`?

[Stack Overflow](https://stackoverflow.com/questions/1177594/when-should-i-use-struct-vs-openstruct) - [OpenStruct Docs](https://ruby-doc.org/stdlib-2.0.0/libdoc/ostruct/rdoc/OpenStruct.html)

> With an `OpenStruct`, you can arbitrarily create attributes. A `Struct`, on the other hand, must have its attributes defined when you create it.
> 
> The choice of one over the other should be based primarily on whether you need to be able to add attributes later.


## Difference between exec, system and %x() or Backticks

[Stack Overflow](https://stackoverflow.com/questions/6338908/ruby-difference-between-exec-system-and-x-or-backticks)

**system**

> The `[system](http://www.ruby-doc.org/core/Kernel.html#method-i-system)` method calls a system program. You have to provide the command as a string argument to this method 

**backticks**

> [Backticks](http://www.ruby-doc.org/core/Kernel.html#method-i-60) (``) call a system program and return its output. As opposed to the first approach, the command is not provided through a string, but by putting it inside a backticks pair.

**%x()**

> Using `%x` is an alternative to the backticks style. It will return the output, too.

**exec**

> By using `[Kernel#exec](http://www.ruby-doc.org/core/Kernel.html#method-i-exec)` the current process (your Ruby script) is replaced with the process invoked through `exec`. The method can take a string as argument. In this case the string will be subject to shell expansion. When using more than one argument, then the first one is used to execute a program and the following are provided as arguments to the program to be invoked.

