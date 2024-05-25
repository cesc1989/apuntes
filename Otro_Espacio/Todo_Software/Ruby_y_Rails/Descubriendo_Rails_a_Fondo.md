# Descubriendo Rails a Fondo
Clases, métodos, módulos y todo aquello que descubro de Rails al trabajar en Luna.

# Funcionamiento de `nested_attributes_for`

*Pendiente por examinar*
Análisis, no tan a fondo, de cómo funciona este método de clase(macro) del módulo `ActiveRecord::NestedAttributes::ClassMethods`.

[Código fuente en GitHub](https://github.com/rails/rails/blob/b9ca94caea2ca6a6cc09abaffaad67b447134079/activerecord/lib/active_record/nested_attributes.rb#L333)

    def accepts_nested_attributes_for(*attr_names)
      options = { allow_destroy: false, update_only: false }
      options.update(attr_names.extract_options!)
      options.assert_valid_keys(:allow_destroy, :reject_if, :limit, :update_only)
      options[:reject_if] = REJECT_ALL_BLANK_PROC if options[:reject_if] == :all_blank
    
      attr_names.each do |association_name|
        if reflection = _reflect_on_association(association_name)
          reflection.autosave = true
          define_autosave_validation_callbacks(reflection)
    
          nested_attributes_options = self.nested_attributes_options.dup
          nested_attributes_options[association_name.to_sym] = options
          self.nested_attributes_options = nested_attributes_options
    
          type = (reflection.collection? ? :collection : :one_to_one)
          generate_association_writer(association_name, type)
        else
          raise ArgumentError, "No association found for name `#{association_name}'. Has it been defined yet?"
        end
      end
    end
# Descubriendo `ActiveRecord::Reflection::ClassMethods`

[Ver docs de](https://api.rubyonrails.org/classes/ActiveRecord/Reflection/ClassMethods.html) `[ActiveRecord::Reflection::ClassMethods](https://api.rubyonrails.org/classes/ActiveRecord/Reflection/ClassMethods.html)`.

Explicación inicial según los docs:

> Reflection enables the ability to examine the associations and aggregations of Active Record classes and objects

Con los métodos disponibles en este módulo se puede obtener todas las asociaciones de un modelo y trabajar sobre ellas.

Por ejemplo, para el caso del endpoint `SubmitForm` del *credentialing form*, usando `Therapist.reflect_on_all_associations(:has_one)` obtengo todas las asociaciones del tipo especificado y luego poder trabajar sobre sus instancias.

Un ejemplo más concreto

    assocs = Therapist.reflect_on_all_associations(:has_one).map { |assoc| assoc.name }
    
    t = Therapist.find("76b3b568-31b7-48e6-a4b9-3fa20772b0a5")
    
    ers = assocs.each_with_object({}) do |assoc, memo|
      relation = t.send(assoc)
    
      next unless relation
    
      if relation.invalid?
        memo[relation.model_name.singular] = relation.errors.messages
      end
    end
    
    p ers # retorna hash de hashes con los errores por cada asociación encontrada e inválida
# Descubriendo `ActiveRecord::Aggregations::ClassMethods`

*Pendiente por examinar*
[Ver docs de](https://api.rubyonrails.org/classes/ActiveRecord/Aggregations/ClassMethods.html) `[ActiveRecord::Aggregations::ClassMethods](https://api.rubyonrails.org/classes/ActiveRecord/Aggregations/ClassMethods.html)`.

Explicación inicial según los docs:

> Active Record implements aggregation through a macro-like class method called [composed_of](https://api.rubyonrails.org/classes/ActiveRecord/Aggregations/ClassMethods.html#method-i-composed_of) for representing attributes as value objects. It expresses relationships like “Account [is] composed of Money [among other things]” or “Person [is] composed of [an] address”. Each call to the macro adds a description of how the value objects are created from the attributes of the entity object (when the entity is initialized either as a new object or from finding an existing object) and how it can be turned back into attributes (when the entity is saved to the database).
# Descubriendo `concurrent-ruby`
> Modern concurrency tools for Ruby. Inspired by [Erlang](http://www.erlang.org/doc/reference_manual/processes.html), [Clojure](http://clojure.org/concurrent_programming), [Scala](http://akka.io/), [Haskell](http://www.haskell.org/haskellwiki/Applications_and_libraries/Concurrency_and_parallelism#Concurrent_Haskell), [F#](http://blogs.msdn.com/b/dsyme/archive/2010/02/15/async-and-parallel-design-patterns-in-f-part-3-agents.aspx), [C#](http://msdn.microsoft.com/en-us/library/vstudio/hh191443.aspx), [Java](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html), and classic concurrency patterns.

[GitHub del proyecto](https://github.com/ruby-concurrency/concurrent-ruby).

Lo encuentro al ver esto en la consola de rails

    [1] pry(main)> Therapist.reflect_on_association(:payout)
    => #<ActiveRecord::Reflection::HasOneReflection:0x00007fe652a1bb58
     @active_record=Therapist (call 'Therapist.connection' to establish a connection),
     @association_scope_cache=#<Concurrent::Map:0x00007fe652a1ba40 entries=0 default_proc=nil>,
     @constructable=true,
     @foreign_type=nil,
     @klass=nil,
     @name=:payout,
     @options={:dependent=>:destroy},
     @plural_name="payouts",
     @scope=nil,
     @type=nil>

Énfasis en `@association_scope_cache=#<Concurrent::Map:0x00007fe652a1ba40 entries=0 default_proc=nil>`.

Ver `[Concurrent::Map](http://ruby-concurrency.github.io/concurrent-ruby/master/Concurrent/Map.html)`.

Busqué por el lado de Rails y [encontré fue el PR](https://github.com/rails/rails/pull/22185)(del año 2015) que indica el uso de esta clase. Se trata de una librería que ofrece características de concurrencia disponible en otros lenguajes.

# Uso de submodulo `ClassMethods` en varios modulos de Rails

He notado como varias de las gemas que usa Rails definen un submodulo llamado `ClassMethods`. En algún momento se debía al [patrón plugin explicado por Yehuda Katz](https://yehudakatz.com/2009/11/12/better-ruby-idioms/), sin embargo, en versiones más recientes no lo veo tan aplicado como en el artículo.

Se puede apreciar el uso de este “patrón” en los módulos:


- `ActiveRecord::Reflection::ClassMethods`
- `ActiveRecord::Aggregations::ClassMethods` (este está con visibilidad *privada*)
- `ActiveRecord::Core::ClassMethods`
- `ActiveRecord::NestedAttributes::ClassMethods`
- `ActiveRecord::Associations::ClassMethods`

En una pregunta en [Stack Overflow](https://stackoverflow.com/a/30757763/1407371)(hace 4 años) hay una posible explicación:


> This is actually a pretty common practice in Ruby. Basically, what it's saying is: when some object performs `include MyModule`, make it also `extend MyModule::ClassMethods`. Such a feat is useful if you want a mixin that adds some methods not just to the instances of a class, but to the class itself.
## Ruby Pattern: Extend through Include

Una explicación más clara está en el artículo [*Ruby Pattern: Extend through Include*](https://www.dan-manges.com/blog/27). Se explica que esta practica se hace por conveniencia para que al incluir el modulo, también se extienda.


> However, the majority of the time modules are used with classes, **they are included, not extended**. Even if I am writing a module which only adds class methods (and could therefore be used with extend rather than include)

Ejemplo en el artículo:

    module ExtendThroughInclude
      def self.included(klass)
        klass.extend ClassMethods
      end
    
      def instance_method
        "this is an instance of #{self.class}"
      end
    
      module ClassMethods
        def class_method
          "this is a method on the #{self} class"
        end
      end
    end
    
    class Person
      include ExtendThroughInclude
    end
    
    Person.new.instance_method #=> "this is an instance of Person"
    Person.class_method #=> "this is a method on the Person class"
## Gema SuperModule

También existe una gema cuyo fin es permitir dejar a un lado el uso de este patrón, se llama [SuperModule](https://github.com/AndyObtiva/super_module). [Artículo con explicación](https://www.airpair.com/ruby/posts/step-aside-activesupportconcern-supermodule-is-the-new-sheriff-in-town).


> Tired of [Ruby](https://www.ruby-lang.org/en/)'s modules not allowing you to mix in class methods easily? Tired of writing complex `self.included(base)` code or using over-engineered solutions like `[ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)` to accomplish that goal?
> 
> [SuperModule](https://rubygems.org/gems/super_module) allows defining class methods and method invocations the same way a super class does without using `[self.included(base)](http://ruby-doc.org/core-2.2.1/Module.html#method-i-included)`.
> 
> This succeeds `[ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)` by offering lighter syntax and simpler module dependency support.

