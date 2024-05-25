# Cap 13. Comments should describe things that aren’t obvious from the Code

> The reason for writing comments is that statements in a programming language can’t capture all of the important information that was in the mind of the developer when the code was written.


> The guiding principle for comments is that comments should describe things that aren’t obvious from the code.


> The idea of an abstraction is to provide a simple way of thinking about something, but code is so detailed that it can be hard to see the abstraction just from reading the code.

Even if this information can be deduced by reading the code, we don’t want to force users of a module to do that: reading the code is time-consuming and forces them to consider a lot of information that isn’t needed to use the module.


> **Developers should be able to understand the abstraction provided by a module without reading any code other than its externally visible declarations.**


> Conventions serve two purposes. First, they ensure consistency, which makes comments easier to read and understand. Second, they help to ensure that you actually write comments.


## Categorías para comentarios

**Interface**
a comment block that immediately precedes the declaration of a module such as a class, data structure, function, or method.

**Data structure member**
a comment next to the declaration of a field in a data structure,

**Implementation comment**
a comment inside the code of a method or function, which describes how the code works internally.

**Cross-module comment**
a comment describing dependencies that cross module boundaries.

Clave esto

> **it is easier to comment everything rather than spend energy worrying about whether a comment is needed.**
----------


> After you have written a comment, ask yourself the following question: could someone who has never seen the code write the comment just by looking at the code next to the comment?


> A first step towards writing good comments is to use different words in the comment from those in the name of the entity being described.


## Lower-level comments add precision
> Comments augment the code by providing information at a different level of detail.


> Some comments provide information at a lower, more detailed, level than the code; these comments add precision by clarifying the exact meaning of the code.


> provide information at a higher, more abstract, level than the code; these comments offer intuition, such as the reasoning behind the code, or a simpler and more abstract way of thinking about the code.


> When documenting a variable, think nouns, not verbs. In other words, focus on what the variable represents, not how it is manipulated.


## Higher-level comments enhance intuition
> Higher-level comments are more difficult to write than lower-level comments because you must think about the code in a different way. Ask yourself: What is this code trying to do? What is the simplest thing you can say that explains everything in the code? What is the most important thing about this code?


> But, great software designers can also step back from the details and think about a system at a higher level. This means deciding which aspects of the system are most important, and being able to ignore the low-level details and think about the system only in terms of its most fundamental characteristics.

**This is the essence of abstraction (finding a simple way to think about a complex entity).**


## Interface documentation
> If you want code that presents good abstractions, you must document those abstractions with comments.


> Interface comments provide information that someone needs to know in order to use a class or method;


> Implementation comments describe how a class or method works internally in order to implement the abstraction.


## Implementation comments: what and why, not how
> The main goal of implementation comments is to help readers understand what the code is doing (not how it does it).


> If all of the uses of a variable are visible within a few lines of each other, it’s usually easy to understand the variable’s purpose without a comment. In this case it’s OK to let readers read the code to figure out the meaning of the variable.


## Cross-module design decisions
> I have recently been experimenting with an approach where cross-module issues are documented in a central file called designNotes.

Como aquello de documentar “design decisions”.

— 


> The goal of comments is to ensure that the structure and behavior of the system is obvious to readers, so they can quickly find the information they need and make modifications to the system with confidence that they will work.


> When writing comments, try to put yourself in the mindset of the reader and ask yourself what are the key things he or she will need to know.

Y lo más importante de documentar:

> **if a reader thinks it’s not obvious, then it’s not obvious.**

