# Cap.6 - General-Purpose Modules are Deeper

> I now think that over-specialization may be the single greatest cause of complexity in software.
> 
> Conversely, code that is more general-purpose is simpler, cleaner, and easier to understand.


## Make classes somewhat general-purpose
> we know that it’s hard to predict the future needs of a software system, so a general-purpose solution might include facilities that are never actually needed.
> 
> Furthermore, if you implement something that is too general-purpose, it might not do a good job of solving the particular problem you have today.

Que sea génerico por justo para resolver el problema de hoy:

> In my experience, the sweet spot is to implement new modules in a somewhat general-purpose fashion. The phrase “somewhat general-purpose” means that the module’s functionality should reflect your current needs, but its interface should not.


> The interface should be easy to use for today’s needs without being tied specifically to them.


## Generality leads to better information hiding
> One of the most important elements of software design is determining who needs to know what, and when. When the details are important, it is better to make them explicit and as obvious as possible

**Is this API easy to use for my current needs?**

> If you have to write a lot of additional code to use a class for your current purpose, that’s a red flag that the interface doesn’t provide the right functionality.
## Conclusion
> Specialization can’t be eliminated completely, but with good design you should be able to reduce it significantly and separate specialiazed code from general-purpose code.
> 
> This will result in deeper classes, better information hiding, and simpler and more obvious code.

