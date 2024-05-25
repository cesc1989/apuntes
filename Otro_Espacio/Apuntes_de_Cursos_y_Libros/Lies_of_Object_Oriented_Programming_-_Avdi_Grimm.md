# Lies of Object Oriented Programming - Avdi Grimm

# Email 1

The idea that object oriented programming has failed is false. Pointless.


## A philosophical case for OOP
> At its core, object oriented programming is not a school of programming language design. **OO is a form of** **applied philosophy**. It's a way of looking at the world, and at problems to be solved.

Humans intereact with tangible and untangible things.


> Human intellectual existence is a constant dance of perceiving and inventing objects. And we are only able to accomplish work together because **we come to agreements about the existence of objects: including the ones which have no physical existence.**

OOP is a way of:

1. Explicitly acknowledging this truth of human existence, and
2. Enlisting computers to help us imagine, agree on, and communicate about the objects which make up our working world.


> When object oriented programming was first conceived by Alan Kay in the 1970s, it was a deliberate shift in perspective from "how computers work", **to "how humans think"**.


# Email 2
> But I believe that **every programming language has a "grain"**, much like a wooden log has a grain. You can code "with the grain" and find your work easy and natural. **Or you can code "against the grain" and find yourself fighting the system**.

Programming ecosystem have a set of tools and practices that work best when used all together. What are some good examples of this?


- Ruby ecosystem: Rails, Bundler, rbenv or RVM, Devise, Sidekiq, RSpec.
- JavaScript ecosystem: yarn, modern framework.
- DevOps ecosystem: cloud provider managed with Terraform.

Whatever programming language you use, best is to learn to “code with the grain”. 


# Email 3

Here Avdi talks about service objects and having those classes that look like:

    IpnProcessor.new.process_ipn

I also think that’s very redundant but have kept doing that way.


> This kind of class-name/method-name reiteration is always a red flag for me in terms of domain modeling. Sandi Metz, author of Practical Object-Oriented Design in Ruby, says to first identify the message, and then identify a fitting role to receive that message. Saying that a `process_ipn` message should be received by an "IPN processor" seems tautological.

It’s obvious the ipn_processor is the one that can receive the `process_ipn` message.


> It's a bit of a cheat, in fact. We could say the same for any message: "who should receive the `save` message? Why, the `Saver`, of course! What about the `increment_amount` message? The `AmountIncrementer`!"

When I write these kind of classes in the end they are a procedure. One way to organize them is in a module.


## Service Object to Procedure

The wrong object causes more trouble than no object at all.


> An object that handles business logic but doesn't have a well-defined business domain role is [causing trouble down the road]


> Objects tend to grow and accumulate more responsibilities. As Corey Haines puts it, **objects are** **attractors** **for functionality**. And once they mature, **objects with confused, ill-defined roles can be some of the hardest to refactor**.

Service objects accumulate invisible inter-service coupling.

One of the dangers of service objects is when developers start mixing them and there’s no clear indication of the connection.


> you can end up with a whole basket full of Service Objects, many with implicit data dependencies between them, representing business workflows that have no explicit representation. In the worst cases, these pseudo-independent objects don't just share database tables, but also pass state to each other as data-blob hashes via the user session or via URL variables.


## What Domain-Drive Design says
> DDD deliberately avoids using the term "Service Objects", instead simply calling them "Services"


- **Most services should be** ***infrastructure*** **services, such as "send an email"**. Business domain services should be rare.
- As a corollary, infrastructure-level and domain-level services should be kept *separate*.
- **Services should have no persistent state**.


> All of these guidelines are consistent with representing Services as procedures in a Ruby application.


# Email 4
> Objects aren't data plus code. Objects are about behavior. The more we embrace this idea, the more we are coding "with the grain" of an Object-Oriented language.


# Email 5

On the importance of naming things well.

## The power of a name


> There is a misconception at the root of the conventional wisdom about object-oriented programming (…)  **It's the idea that object-oriented programming is principally about** **objects**.


> I'm sorry that I long ago coined the term "objects" for this topic because it gets many people to focus on the lesser idea... **The big idea is "messaging".**
> 
> The key in making great and growable systems is much more to design how its modules communicate rather than what their internal properties and behaviors should be.
> 
>   - Alan Kay


## Four qualities of a good message
> OOP isn't about language technologies. It's about your perspective. And just because an object has a method to be called, doesn't mean you've enabled message-sending semantics in your design.

**Messages are late-bound**
Only the recipient of the message should decide what to do with it.

**Messages are discretionary**

> If the recipient of a message should get to decide what to do about that message, it follows that one possible options for handling the message is to **do nothing.**

**Messages are one-way**
Traditionally in OO languages, messages are implemented as call-and-return values.

With the right perspective and coding strategies you can learn to treat more object methods as true one-way messages. See [East-Oriented Code](https://www.saturnflyer.com/blog/the-4-rules-of-east-oriented-code-rule-1?__s=etpycwt58v8phxbsfbqt).

**Messages use commoditized formats**

> well-designed messages often pass information in a format that's "commoditized" for easy interchange. For instance, an `EmailParser` class might use a complex `EmailAddress` type internally for representing email addresses. But when it sends a message to collaborating object, it might pass the email addresses as simple strings. That way, it isn't requiring other objects to know about the `EmailAddress` type. ****


> For instance, within the domain models of an ecommerce site, it might be common to pass a homegrown `Money` type around from object to object. But when communicating with, e.g. the view layer, it may be necessary to "drop down" to a more basic level of commoditized data.


# Email 6
> The truth is, **objects are for mental modeling, not simulation**. They exist to help us build up a shared, animated "reality" of concepts **which** **don't** **have easy analogs in the physical world**. Things like car warranties. And maintenance histories. And rental agreements.


> The area I see this "simulation fallacy" bite developers the most is when it tempts them into simulating people. The result is the ubiquitous `User` class, **which so often grows to thousands of line in typical Ruby on Rails applications**.

objects are great at representing *relationships*. I'm talking about relationships between people and other entities, like:

    Employee
    Shareholder
    DepartmentHead
    Student
    GroupMember
    Licensee

These relationships, which exist only as agreements between humans, are exactly the kind of concepts that are most in need of consensus and coordination. And they are the kind of thing that software objects excel at representing.

# Email 7

On inheritance.


> There are very few true IS-A relationships in the real world.
> 
> - Dave Thomas

To my understading, Ruby didn’t support multiple inheritance. Turns out, according to Avdi:

> Ruby is a multiple-inheritance programming language.
> 
> Ruby modules are a multiple inheritance mechanism.
> 
> And in fact, Ruby's class inheritance is really just a glorified form of module inheritance.

Avdi advocates for Composition over inheritance.

# Email 8

Look for and try to find those objects between objects already working together.


> Often, when I encounter codebases with large "god object" business models, **it's because they are having to stand in for numerous unrecognized business domain concepts**.
> 
> When trying to figure out where some logic belongs, try to look beyond the "filing cabinet objects". Look to see if there are missing, implicit concepts that deserve their own objects. Concepts such as:


    Rules for making important decisions
    Business processes or workflows that tie other objects into a narrative.
    Relationships between existing objects
    Placeholders for missing or to-be-determined information.
    Specialized collections of other objects.
    Perspectives on existing models which are only applicable to a given context, e.g. an auditor's view of a bank transaction history.


# Email 9

Classes or premmature classes are no obligatory.


# Email 10


