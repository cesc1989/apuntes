# Teams should be an MVP Feature

El artículo [Teams should be an MVP feature](https://blog.bullettrain.co/teams-should-be-an-mvp-feature/) explica que, al crear nuevas aplicaciones Ruby on Rails, se debe soportar desde el día uno un modelo `Team` u `Organization` que centralice los recursos creados en el sistema y así permitir que varios usuarios accedan a dichos recursos.

El modelo `Organization` va en contra parte a lo que solemos hacer con frecuencia donde se crea un modelo `User` el cual será el dueño de todos los recursos que se crean en el sistema/plataforma.

Dejar que el modelo `User` sea el dueño de todo tiene varias limitantes como:

- Tener que compartir credenciales para delegar tareas a otras personas.
- Poder usar la plataforma como ambos lados de la relación.
	- Tocaría crear una cuenta de conductor y otra de pasajero en InDriver, por ejemplo.
- Igual se necesita un modelo `Organization` en caso de que empresas quieran pagar por el uso del servicio a sus empleados.

Implementar este modelo `Organization` es una de esas cosas que duele más hacerlas en el futuro que a inicios del proyecto.

> who have skipped implementing teams in an application early on and then found ourselves having to implement it later can attest to how challenging it can be

Al final, tiene más sentido que sea un `Organization` quien sea dueña de los recursos:

> my experience is that most software applications should model most of the primary components of their domain model as belonging to the joint team or organization

El autor del artículo explica como está formado este modelo en [Bullet Train](https://bullettrain.co/) (una plantilla Ruby on Rails):

- A `User` belongs to a `Team` through a `Membership`.
- The `Membership` can have one or more `Role`s associated with it.
- The `Membership` model itself can be extended by developers to include additional types of permissions that can be configured on a per-`Membership` basis, e.g. access to certain features or specific resources.
- When adding a new `Membership`, an `Invitation` is also created and sent via email.
- The `Invitation` can be claimed by either a new or existing `User`, even if the email address doesn’t match where the invitation was delivered too.
- The `Invitation` goes away once claimed.

## Modelo Membership y Organization en Bullet Train

Vayamos al código y veamos.