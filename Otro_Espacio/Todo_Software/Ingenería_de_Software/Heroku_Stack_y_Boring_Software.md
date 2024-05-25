# Heroku Stack y Boring Software
El *Heroku Stack* hace referencia a NO usar herramientas y servicios que requieran conocimientos técnicos muy avanzados, ejemplo Terraform o Kubernetes, para una aplicación recién empezada.

- [You Are Not Google](https://blog.bradfieldcs.com/you-are-not-google-84912cf44afb)
- Correo de Ryan de Booster Stage
## The Heroku Stack
> I stumbled on this tweet this morning (please excuse the language)…
> The Kamal stack is:
> 
> K - you don't have
> A - product market fit yet
> M - why you fucking around
> A - with kubernetes
> L - just use heroku
![](https://paper-attachments.dropbox.com/s_3B81EBFF5CDEE9B48C4ECB1936F33513372D69140FB3F40FBE90ADDF4A521033_1573085053576_image.png)

> [https://twitter.com/kamal/status/1091020666206179328](https://twitter.com/kamal/status/1091020666206179328)
> 
> We get asked all the time what tech stack  we use, and I always find my self justifying Heroku. For many people,  Heroku doesn’t feel like a “serious” hosting provider.
> 
> But the fact is that Heroku is the safest and least risky tech stack for startups.
> When startups bleed money, they usually do it in two ways:
> 
    1. Overspending on technology.
    2. Overbuilding the product.
> 
> I’ll talk about overbuilding the product another time – that’s a huge issue.
> 
> Overspending on technology is tempting  because, well let’s face it, entrepreneurs are notorious optimists. You  wouldn’t be doing this if you didn’t expect customers to bang down your  door for access to your product. The temptation is to overbuild in  anticipation of this overwhelming traffic.
> 
> **The trouble comes from overspending on technology before you’ve achieved.**
> 
> Server resources are expensive, but Human Resources are even more so. Provisioning AWS servers, Docker clusters,  and Kubernetes instances can easily become a full-time job for more than  one person.
> 
> It is true that once you do achieve  product-market fit and the floodgates do open, Heroku may become  unsustainably expensive. In the meantime, Heroku allows you to scale  your server usage up and down based on demand. But most importantly, it  allows you to launch and grow a fully-fledged and professional web  product without devoting expensive engineering resources to managing the  details of your tech stack.

Por su parte, *Boring Software* hace alusión a usar lenguajes de programación y/o frameworks maduros y estables y no seguirse tanto por la moda de usar todo lo nuevo que vaya saliendo, ejemplo cada nuevo *framework* o librería en JS que sale.

- [Choose Boring Technology](http://boringtechnology.club/)


## [The "Boring Software"](https://tqdev.com/2018-the-boring-software-manifesto) [M](https://tqdev.com/2018-the-boring-software-manifesto)[anifesto](https://tqdev.com/2018-the-boring-software-manifesto)
> That is, in pursuit of "agility and craftsmanship" we have found "boring software" to be indispensable.

Lo que está en la izquierda es estable. Lo que está en la derecha sobrevalorado.

- 3-tier applications vs. micro services
- relational database vs. NoSQL
- page reloads vs. single page applications


## [Choose Boring Technology](http://mcfunley.com/choose-boring-technology)
> **Embrace Boredom**

Cada compañía tiene unos tokens de innovación. Tres. Los cuales deben gastarse con cuidado para no perder el enfoque de lo que se quiere construir. ¿Vas a construir una aplicación que es muy CRUD? Usa Rails cuyo fuerte es este tipo de software.


> What counts as boring? That’s a little tricky. “Boring” should not be conflated with “bad.”

Que sea aburrido no significa que sea malo, aunque sí hay lo que es aburrido y malo no todo es así: MySQL, PHP, Ruby, Python.


> The nice thing about boringness (so constrained) is that the capabilities of these things are well understood. But more importantly, their failure modes are well understood.

Son tan aburridos que la mayoría de errores ya están descubiertos y resueltos. Ejemplo, Rails.


> But for shiny new technology the magnitude of unknown unknowns is significantly larger, and this is important.

El problema de “aquello que sé que no sé” vs “aquello que no sé que no sé”. Lo que sé que desconozco vs lo que no sé que desconozco. Para las tecnologías nuevas, la magnitud de lo “que no sé que desconozco” es enorme y puede ser peligrosa.


> **Choose new techonology, sometimes**

Cuando esto pase, debe ser un consenso de grupo y no una decisión individual.

Vale la pena primero intentar resolver el problema con lo que se tiene a mano. De esta forma se puede descartar traer una tecnología nueva solo por gusto.


## [Build](https://www.intercom.com/blog/videos/build-boring-software) [B](https://www.intercom.com/blog/videos/build-boring-software)[oring](https://www.intercom.com/blog/videos/build-boring-software) [S](https://www.intercom.com/blog/videos/build-boring-software)[oftware](https://www.intercom.com/blog/videos/build-boring-software)
> When trying to solve a problem with software, it can be tempting to build an exciting, complex solution. But that unnecessary complexity can lead to even more complicated issues.


> a boring problem is one that’s well understood.


> You can’t expect to come up with a solution if you don’t understand the problem.

If you think about adding a new technology to your stack, you should ask yourself these five questions:


1. How do we deploy it?
2. How do we maintain it?
3. How do we train people to use it?
4. How do we recover when it fails?
5. How do we develop with it locally?


> “If you’re as clever as you can be when you write it, how will you ever debug it?”
> “Being clever is dumb.”

When you’re trying to solve a problem, you want to keep that capacity for solving the problem, not remembering a lot of stuff. Here are 3 tips on how to do that:

- **Use patterns:** We’re good at recognizing patterns. It means that you can push that down to the lower level stuff and you don’t have to remember what every other thing does.
- **Be explicit:** Remove levels of indirection, because it just means it’s less context that you need to remember.
- **Be consistent:** We have this concept of a signal, and everybody on our team, front end, back end, PM and design, all call it a signal. It just means my brain doesn’t have to map something to something else, and then onto something else. I can just get on with solving the problem.


> the definition of boring should be: “easy to understand, familiar and uneventful.”


## [Help! None of my projects want to be SPAs](https://whatisjasongoldstein.com/writing/help-none-of-my-projects-want-to-be-spas/)

Mención a “[Magpie Developer](https://blog.codinghorror.com/the-magpie-developer/)” the Jeff Atwood.

React → bueno para proyectos con bastante flujo de información: Jira, Asana. Malo para sitios de noticias.

Si es CRUD, no te compliques. Usa Rails o Django.

I can think of a few things that would benefit from being SPAs. A chat application is a no-brainer. A dashboard of realtime data might not need to be a SPA, but would benefit from a reactive component that updates when the API is polled. Maybe a complicated tool like Trello, a Calendar or a Nonlinear Video Editor could make sense, but even those have distinct screens, **so a large reactive component on a particular screen may be a better fit**.

