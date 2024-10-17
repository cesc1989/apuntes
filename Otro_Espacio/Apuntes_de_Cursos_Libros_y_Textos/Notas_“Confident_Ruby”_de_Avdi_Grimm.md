# Notas “Confident Ruby” de Avdi Grimm
Las notas están en [este documento](https://docs.google.com/document/d/1YYHtIlYdfkLLmqWhcnlG-h54Y9LIhznYnv3yR8dispg/edit).


> methods that tell a good story lump these four parts of a method together in distinct "stripes", rather than mixing them will-nilly. But not only that, they do it in the order I listed above: First, **collect input**. Then **perform work**. Then **deliver output**. Finally, **handle failure, if necessary**.


1. Collect input
2. Perform work
3. Deliver output
4. Handle failure
# Performing Work
> The decisions we make as object-oriented programmers are about what messages are sent, under which circumstances, and which objects will receive them.

Lo que permite que:


> A captain with a well-trained crew doesn't waste time checking her subordinates' insignia to see if they are qualified to execute an order; asking them if they understood the command, or giving them detailed instructions about how to do their job.


## Identificando Roles
> What's a role? Rebecca Wirfs-Brock calls it "a set of related responsibilities"

En todo caso:

> However, **a role is not the same concept as a class**: more than one type of object may play a given role, and in some cases a single object might play more than one role.


## Sobre duck typing

Este código hace chequeo de tipos y “está mal”:

    @logger && @logger.info "Imported purchase ID #{purchase_record.id}"


> This last practice is particularly insidious. Make no mistake: NilClass is just another type. Asking an object if it is `nil?`, even implicitly (as in `duck && duck.quack`, or a Rails-style `duck.try(:quack)`), is type-checking just as much as explicitly checking if the object's class is equal to NilClass.


> As confident coders, we want to tell our ducks to quack and then move on


# Collecting Input

Sobre esta porción de código:

    class TimeCalc
      def initialize
        @start_date = Time.now
      end
    
      def time_n_days_from_now(num_days)
        @start_date + num_days * 24 * 60 * 60
      end
    end


>  We don't always think of class names as inputs. But in a language where classes are just another kind of object, and class names are just ordinary constants that happen to refer to a class, an explicit class name is an input like any other. It's information that comes from outside the method.


## Indirect Inputs
> Any time we send a message to an object other than self in order to use its return value, we're using indirect inputs.


> The more levels of indirection that appear in a single method, the more closely that method is tied to the structure of the code around it. And the more likely it is to break, if that structure changes


## Use built-in conversion protocols

**Indications**
You want to ensure that inputs are of a specific core type. For instance, you are writing a method with logic that assumes Integer inputs.

**Synopsis**
Use Ruby's defined conversion protocols, like `#to_str`, `#to_i`, `#to_path`, or `#to_ary`.

**Rationale**
By typing a few extra characters we can ensure that we only deal with the types we expect, while providing greater flexibility in the types of inputs our method can accept.

**Explicit and Implicit Conversions**

> `#to_s` is an explicit conversion method. Explicit conversions represent conversions from classes which are mostly or entirely unrelated to the target class.


> `#to_str`, on the other hand, is an implicit conversion method. Implicit conversions represent conversions from a class that is closely related to the target class.

The "implicit"/"explicit" terminology stems from how the Ruby language and core classes interact with these methods. In some cases Ruby classes will implicitly send messages like `#to_str` or `#to_ary` to objects they are given, in order to ensure they are working with the expected type. By contrast, we have to explicitly send messages like `#to_s` and `#to_a`—Ruby won't automatically use them.


    now = Time.now
    
    now.respond_to?(:to_s)          # => true
    now.to_s                        # => "2013-06-26 18:42:19 -0400"
    
    now.respond_to?(:to_str)        # => false


> Explicit conversions are there for you, the programmer, to use if you know that you want to tell one type to convert itself to another, even if the types are very different

Por ejemplo:

    Time.now.to_s                   # => "2013-06-26 19:09:15 -0400"
    "1 ton tomato".to_i             # => 1
    {x: 23, y: 32}.to_a             # => [[:x, 23], [:y, 32]]

**If you know what you want, ask for it**

> Just as Ruby core classes use `#to_str` and other methods to ensure they have a inputs they can use, we can benefit from the practice of calling conversion methods as well.

En vez de asumir que algún valor de ingreso o método es de algún tipo principal como String, Integer, Array o Hash, mejor deberíamos asegurarnos que sean de ese tipo haciendo una conversión explícita.


>  We need an Integer? Use `to_i` or `to_int!` We need a String? Use `to_s` or `to_str!` If we need a Hash and we're using Ruby 2.0, send  `to_h!`


## Reject unworkable values with preconditions

**Indications**
Some input values to a method cannot be converted or adapted to a usable form. Accepting them into the method has potentially harmful or difficult-to-debug side effects. Example: a hire_date of nil for an Employee object may result in undefined behavior for some Employee methods.

**Synopsis**
Reject unacceptable values early, using precondition clauses.

**Rationale**
It is better to fail early and obviously than to partially succeed and then raise a confusing exception.


> One of the purposes of a constructor is to establish an object's invariant: a set of properties which should always hold true for that object

O sea que hay que validar las variables que estén en el constructor de la clase.

En vez de:


      def initialize(name, hire_date)
        @name      = name
        @hire_date = hire_date
      end  

Mejor:

      def initialize(name, hire_date)
        @name          = name
        self.hire_date = hire_date
      end
    
      def hire_date=(new_hire_date)
        raise TypeError, "Invalid hire date" unless new_hire_date.is_a?(Date)
        @hire_date = new_hire_date
      end

Hacer esto tiene dos beneficios:

1. Protegen al método de valores inválidos que podrían afectar el programa.
2. Se vuelven documentación ejecutable.


> because of their prominent location at the start of a method, they serve as executable documentation of the kind of inputs the method expects. When reading the code, the first thing we see is the precondition clause, telling us what values are out of bounds for the method.

Además:

> More insidiously, code written in an atmosphere of fear and doubt about bad inputs can lead to multiple, inconsistent ways of handling unexpected values being developed


## Document assumptions with assertions

**Indications**
A method receives input data from an external system, such as a feed of banking transactions. The format of the input is under-documented and potentially volatile.

**Synopsis**
Document every assumption about the format of the incoming data with an assertion at the point where the assumed feature is first used. Use assertion failures to improve your understanding of the data, and to warn you when its format changes.

**Rationale**
When dealing with inconsistent, volatile, under-documented inputs from external systems, copious assertions can both validate our understanding as well serve as a "canary in a coal mine" for when the inputs change without warning.

Sobre este código: Example: Importing bank transactions

    class Account
      def refresh_transactions
        transactions = bank.read_transactions(account_number)
        transactions.is_a?(Array) or raise TypeError, "transactions is not an Array"
        transactions.each do |transaction|
          amount = transaction.fetch("amount")
          amount_cents = (Float(amount) * 100).to_i
          cache_transaction(:amount => amount_cents)
        end
      end
    end

Hay mucha verificación de tipo lo cual es algo que el libro sugiere no hacer pero en este caso como es un sistema externo del cual sabemos poco o no tenemos total control, es mejor prevenir que lamentar. Mejor curarse en salud.


> This code clearly states what it expects. It communicates a great deal of information about our understanding of the external API at the time we wrote it. It explicitly establishes the parameters within which it can operate confidently…and as soon as any of its expectations are violated it fails quickly, with a meaningful exception message.


> By failing early rather than allowing misunderstood inputs to contaminate the system, it reduces the need for type-checking and coercion in other methods.

El duck-typing (confiar en que los valores de ingreso son del tipo que esperamos) es posible cuando se tiene algún control sobre los datos de ingreso. Sea que los creemos nosotros mismos o tenemos comunicación con el equipo que los genera.

Pero en casos de no tener comunicación ni control:

> But sometimes we have to communicate with systems we have no control over, and for which documentation is lacking. We may have little understanding of what form input may take, and even of how consistent that form will be. At times like this, it is better to state our assumptions in such a way that we know immediately when those assumptions turn out to be false.


## Represent do-nothing cases as null objects

**Indications**
One of the inputs to a method may be nil. The presence of this nil value flags a special case. The handling of the special case is to do nothing—to ignore the input and forgo an interaction which would normally have taken place. For instance, a method might have an optional logger argument. If the logger is nil, the method should not perform logging.

**Synopsis**
Replace the nil value with a special Null Object collaborator object. The Null Object has the same interface as the usual collaborator, but it responds to messages by taking no action.

**Rationale**
*Representing a special "do nothing" case as an object in its own right eliminates checks for nil**. It can also provide a way to quickly "nullify" interactions with external services for the purpose of testing, dry runs, "quiet mode", or disconnected operation.*


    def create_widget(attributes={}, data_store=nil)
      data_store ||= NullObject.new
      Actual(data_store.store(Widget.new(attributes)))
    end


# Delivering Results
> And just as we strive to make the implementation of our own methods clear and confident, we should also ensure that our outputs make it easy for the clients of our code to be written in a confident style as well


## Write total functions

**Indications**
A method may have zero, one, or many meaningful results.

**Synopsis**
Return a (possibly zero-length) collection of results in all cases.

**Rationale**
Guaranteeing an Array return enables calling code to handle results without special case clauses.

Retorna funciones totales: siempre devuelve algo en lugar de nil

> In math, a total function is a function which is defined for all possible inputs. For our purposes in Ruby code, we'll define a total function as a method which never returns nil no matter what the input.


    def find_words(prefix)
      words = File.readlines('/usr/share/dict/words').map(&:chomp).reject{|w| w.include?("'") }
      
      words.select{|w| w =~ /\A#{prefix}/}
    end
    
    find_words('gnu')               # => ["gnu", "gnus"]
    find_words('ruby')              # => ["ruby"]
    find_words('fnord')             # => []


## Callback instead of returning

**Indications**
Client code may need to take action depending on whether a command method made a change to the system.

**Synopsis**
Conditionally yield to a block on completion rather than returning a value.

**Rationale**
A callback on success is more meaningful than a true or false return value.


    # ...
    if import_purchase(date, title, user_email)
      send_book_invitation_email(user_email, title)
    end
    # ...


> This method also violates the principle of command-query separation (CQS). CQS is a simplifying principle of OO design which advises us to write methods which either have side effects (commands), or return values (queries), but never both.

Se sugiere que mejor se haga algo como esto:

    # ...
    import_purchases(purchase_data) do |user, purchase|
      send_book_invitation_email(user.email, purchase.title)
    end
    # ...
## Represent failure with a benign value

**Indications**
A method returns a nonessential value. Sometimes the value is nil.

**Synopsis**
Return a default value, such as an empty string, which will not interfere with the normal operation of the caller.

**Rationale**
Unlike nil, a benign value does not require special checking code to prevent NoMethodError being raised.

Tenemos este método

    def latest_tweets(number)
      # ...fetch tweets...
    rescue Net::HTTPError
      ""
    end

Que se usa acá:

    def render_sidebar
      html = ""
      html << "<h4>What we're thinking about...</h4>"
      html << "<div id='tweets'>"
      html << latest_tweets(3) || ""
      html << "</div>"
    end

Al retornar una cadena vacía en el `rescue` se puede cambiar la línea que usa la función así:

    def render_sidebar
      html = ""
      html << "<h4>What we're thinking about...</h4>"
      html << "<div id='tweets'>"
      html << latest_tweets(3)
      html << "</div>"
    end


> nil is the worst possible representation of a failure: it carries no meaning but can still break things. An exception is more meaningful, but some failure cases aren't really exceptional
## Yield a status object

**Indications**
A command method may have more possible outcomes than success/failure, and we don't want it to return a value.

**Synopsis**
Represent the outcome of the method with a status object with callback-style methods, and yield that object to callers.

**Rationale**
This approach cleanly separates *what* to do for a given outcome from *when* to do it, and even from how often to do it.

Este patrón da una buena forma de cómo usar los bloques. Ver todo el ejemplo en el capítulo.

## Signal early termination with throw

**Indications**
A method needs to signal a distant caller that a process is finished and there is no need to continue. For instance, a SAX XML parser class needs to signal that it has all the data it needs and there is no reason to continue parsing the document.

**Synopsis**
Use `throw` to signal the early termination, and `catch` to handle it.


>  throw and catch work very similarly to exceptions, but unlike exceptions they don't have the connotation that an error has occurred.


    if count_it && @current_length + string.length > @length
      # ...
      throw :done
    end


    catch(:done)
      parser.parse(html) unless html.nil?
    end


> If you find yourself using throw/catch frequently; or if you find yourself looking for excuses to use it, you're probably overusing it.


> And every throw must be executed within a matching catch, or the uncaught throw will cause the program to terminate
# Handling Failure

BRE == begin/rescue/end


> BRE is the pink lawn flamingo of Ruby code: it is a distraction and an eyesore wherever it turns up. There is no greater interruption of the narrative flow of code than to find a walloping great BRE parked smack in the middle of the method's core logic


## Use checked methods for risky operations

**Indications**
A method contains an inline begin/rescue/end block to deal with a possible failure resulting from a system or library call.

**Synopsis**
Wrap the system or library call in a method that handles the possible exception.

**Rationale**
Encapsulating common lower-level errors eliminates duplicated error handling and helps keep methods at a consistent level of abstraction.

En vez de esto:

    def filter_through_pipe(command, message)
      results = nil
      IO.popen(command, "w+") do |process|
        results = begin
        process.write(message)
        process.close_write
        process.read
            rescue Errno::EPIPE
        message
            end
      end
      results
    end

Haz esto:

    def checked_popen(command, mode, error_policy=->{raise})
      IO.popen(command, mode) do |process|
        return yield(process)
      end
    rescue Errno::EPIPE
      error_policy.call
    end
    
    def filter_through_pipe(command, message)
      checked_popen(command, "w+", ->{message}) do |process|
        process.write(message)
        process.close_write
        process.read
      end
    end


> A checked method encapsulates the error case, and centralizes the code needed to handle that case if it ever crops up in another method.
## Use bouncer methods

**Indications**
An error is indicated by program state rather than by an exception. For instance, a failed shell command sets the $? variable to an error status.

**Synopsis**
**Write a method to check for the error state and raise an exception.**

**Rationale**
Like checked methods, bouncer methods DRY up common logic, and keep higher-level logic free from digressions into low-level error-checking.


    def filter_through_pipe(command, message)
      result = checked_popen(command, "w+", ->{message}) do |process|
        process.write(message)
        process.close_write
        process.read
      end
      check_child_exit_status
      result
    end


    def check_child_exit_status
      result = yield
      unless $?.success?
        raise ArgumentError,
        "Command exited with status "\
        "#{$?.exitstatus}"
      end
      result
    end


> A Bouncer Method is a method whose sole job is to raise an exception if it detects an error condition. In Ruby, we can write a bouncer method which takes a block containing the code that may generate the error condition
# Parting Words
> No one sets out to write timid, incoherent code. It arises organically out of dozens of small decisions, decisions we make quickly while trying to make tests pass and deliver functionality.


> Often we make these decisions because there is no immediately obvious alternative.

