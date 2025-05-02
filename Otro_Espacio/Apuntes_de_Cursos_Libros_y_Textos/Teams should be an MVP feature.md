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

# Definición de estos modelos en Bullet Train

> [!Note]
> Se pueden ver en esta página https://github.com/bullet-train-co/bullet_train-core/tree/main/bullet_train/app/models

Vayamos al código y veamos.

Para abrir estos modelos en el editor en local hay que usar el comando que Bulle Train ofrece: `bin/resolve Teams::Base --open`:

```
bin/resolve Teams::Base --open

Absolute path:
  /home/cesc/.gem/ruby/3.2.5/gems/bullet_train-1.19.2/app/models/concerns/teams/base.rb

Package name:
  bullet_train-1.19.2

Opening `/home/cesc/.gem/ruby/3.2.5/gems/bullet_train-1.19.2/app/models/concerns/teams/base.rb`.
```

Todas las relaciones y otros métodos están en concerns.

## Modelo Team

```ruby
module Teams::Base
  extend ActiveSupport::Concern

  included do
    has_many :memberships, dependent: :destroy
    has_many :users, through: :memberships
    has_many :invitations
  end
end
```

## Modelo User

```ruby
module Users::Base
  extend ActiveSupport::Concern

  included do
    has_many :memberships, dependent: :destroy
    has_many :teams, through: :memberships
    has_many :collaborating_users, through: :teams, source: :users

    belongs_to :current_team, class_name: "Team", optional: true
  end
end

```

## Modelo Membership

```ruby
module Memberships::Base
  extend ActiveSupport::Concern

  included do
    belongs_to :user, optional: true
    belongs_to :team
    belongs_to :invitation, optional: true, dependent: :destroy
    belongs_to :added_by, class_name: "Membership", optional: true
    belongs_to :platform_agent_of, class_name: "Platform::Application", optional: true

		validates :user_email, uniqueness: {scope: :team}
  end
end
```

## Modelo Invitations

```ruby
module Invitations::Base
  extend ActiveSupport::Concern

  included do
    belongs_to :team
    belongs_to :from_membership, class_name: "Membership"
    belongs_to :invitation_list, class_name: "Account::Onboarding::InvitationList", optional: true

    has_one :membership, dependent: :nullify

		accepts_nested_attributes_for :membership

		validates :email, presence: true, uniqueness: {scope: :team}

    after_create :set_added_by_membership
    after_create :send_invitation_email

    attribute :uuid, default: -> { SecureRandom.hex }

    def roles
      membership.roles
    end
  end

	def set_added_by_membership
    membership.update(added_by: from_membership)
  end

  def send_invitation_email
    UserMailer.invited(uuid).deliver_later
  end
end
```

## Modelo InvitationList

```ruby
module Account::Onboarding::InvitationLists::Base
  extend ActiveSupport::Concern

  included do
    belongs_to :team
    has_many :invitations
    has_many :memberships, through: :invitations

    accepts_nested_attributes_for :invitations, :memberships
  end
end

```