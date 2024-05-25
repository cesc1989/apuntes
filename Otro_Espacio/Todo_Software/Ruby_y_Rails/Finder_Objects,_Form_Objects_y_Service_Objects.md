# Finder Objects, Form Objects y Service Objects

# Finder Objects

De este [artículo](https://janko.io/finder-objects/) de Janko Marohnic:

> Finder objects are classes which encapsulate querying the database. The idea is that your controllers always query your records through finder objects, never through models directly.


# Form Objects

*Form Objects* para validaciones según el contexto y no en el modelo.

Enlaces:

- [Do You Need That Validation? Let Me Call You Back About It by Tobias Pfeiffer](https://www.youtube.com/watch?v=F05kXVqFWnc)
- [Gema ActionForm](https://github.com/railsgsoc/actionform)
- [On Rails 5, Presenters And Form Objects](https://apotonick.wordpress.com/2015/05/21/on-rails5-presenters-and-form-objects/)
- [Gema Reform](https://github.com/trailblazer/reform)
- [En Reddit de Rails](https://www.reddit.com/r/rails/comments/5ikrxy/good_link_to_a_rails_5_form_object_technique/)
- [ActiveModel Form Objects](https://thoughtbot.com/blog/activemodel-form-objects)


# Sobre Service Objects

*Service Objects* para *callbacks* que no están definidos en el modelo.

Los artículos de Jason Swett, Avdi Grim y Martin Fowler en contra de service objects en OOP.

## Beware of “service objects” in Rails

[Artículo](https://www.codewithjason.com/rails-service-objects/).


> **Service objects throw out the fundamental advantages of object-oriented programming.**
> 
> Instead of having behavior and data neatly encapsulated in easy-to-understand objects with names like `Tweet` or `User`, we have conceptually dubious ideas like `TweetCreator` and `RegisterUser`. “Objects” like this aren’t abstractions of concepts in the domain model. They’re chunks of procedural code masquerading as object-oriented code.


## Enough With the Service Objects Already

[Artículo](https://avdi.codes/service-objects/).


> This kind of class-name/method-name reiteration is always a red flag for me in terms of domain modeling. Sandi Metz, author of [Practical Object-Oriented Design in Ruby](http://amzn.to/2A0dKC5), says to first identify the message, and then identify a fitting role to receive that message. Saying that a `process_ipn` message should be received by an “IPN processor” seems tautological.
> 
> It's a bit of a cheat, in fact. We could say the same for any message: “who should receive the `save` message? Why, the `Saver`, of course! What about the `increment_amount` message? The `AmountIncrementer`!”

Al final, los service objects son programación procedural… En un mundo POO.

> We can see that it doesn't depend on any state in the `IpnProcessor` class.
> 
> And we it seems to walk through various steps, grabbing objects from various places, and performing a sequence of actions on those objects.
> 
> There's a name for this kind of code: a procedure

Los objetos atraen funcionalidad sino se controlan, creceran en el desorden.

> An object that handles business logic but doesn't have a well-defined business domain role is like one of these dangerously-located seedlings. Objects tend to grow and accumulate more responsibilities. As Corey Haines puts it, objects are attractors for functionality. And once they mature, objects with confused, ill-defined roles can be some of the hardest to refactor.

La idea de Service Object viene de DDD pero allí no se llaman así sino “Service”. Principios sugeridos en DDD para crear un Service:

> We should try first to find an appropriate domain model object to receive the functionality, before constructing a service.
> 
> Most services should be infrastructure services, such as “send an email”. Business domain services should be rare. This goes directly against some current trends in Ruby web application development, which advocate for every business action having its own Service Object.
> 
> As a corollary, infrastructure-level and domain-level services should be kept separate.


## Anemic Domain Model

[Artículo](https://martinfowler.com/bliki/AnemicDomainModel.html).


> The fundamental horror of this anti-pattern is that it's so contrary to the basic idea of object-oriented design; which is to combine data and process together. The anemic domain model is really just a procedural style design, exactly the kind of thing that object bigots like me (and Eric) have been fighting since our early days in Smalltalk. What's worse, many people think that anemic objects are real objects, and thus completely miss the point of what object-oriented design is all about.


> Application Layer [his name for Service Layer]: Defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems. The tasks this layer is responsible for are meaningful to the business or necessary for interaction with the application layers of other systems. This layer is kept thin. It does not contain business rules or knowledge, but only coordinates tasks and delegates work to collaborations of domain objects in the next layer down. It does not have state reflecting the business situation, but it can have state that reflects the progress of a task for the user or the program.


> In general, the more behavior you find in the services, the more likely you are to be robbing yourself of the benefits of a domain model. If all your logic is in services, you've robbed yourself blind.


## Martin Fowler on Service Objects via the Ruby Rogues Parley mailing list

[Artículo](https://gist.github.com/blaix/5764401).


> The conceptual problem I run into in a lot of codebases is that rather than representing a process, the "service objects" represent "a thing that does the process".


