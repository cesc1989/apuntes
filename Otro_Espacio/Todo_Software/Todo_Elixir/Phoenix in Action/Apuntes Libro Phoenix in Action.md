# Sobre versiones de Elixir y Phoenix

> Elixir went from version 1.6 to 1.8.0-rc.1, and Phoenix went from version 1.3 to 1.4. The Elixir code should work in any version from 1.5 through 1.8 (and beyond), but the listings were written in version 1.7.

## Configurando con asdf

Se supone que lo había instalado pero cuando intentaba usar elixir daba un error:
```
$ iex -r auction.ex 

No version is set for command iex

Consider adding one of the following versions in your config file at 

elixir 1.15.4-otp-26
```

Se arregló así:
```
$ asdf global elixir 1.15.4-otp-26
```

Vi la solución [aquí](https://stackoverflow.com/questions/73588577/asdf-no-version-is-set-for-command-elixir).

## Atomos

> Atoms start with a colon (:), they’re constants, and they are their own value. For example, :foo can never mean or be more than :foo—**it can’t be rebound, its value (:foo) can’t change**, and what you see is what you get.

## Views en Phoenix

No son como las vistas en Rails. Acá son un paso previo antes de. Son como los _Presenters_ o las _Facades_.

> ==The view is responsible for rendering templates and for setting up helper functions that you can use in those templates==. The functions defined here are similar to decorators or presenters in other frameworks

# Plug: ¿Qué es?

Página web https://hexdocs.pm/plug/readme.html
> A specification for composing web applications with functions

El libro lo explica mejor en el capítulo 10.3

Un Plug es un comportamiento que acepta la conexión de una petición, la transforma y luego la transfiere.

> [!Note]
> La conexión de la petición (connection request) es el struct `Plug.Conn` que se pasa a cada función de un controlador.

## Tipos de Plugs

Hay dos tipos: funciones y módulos.

> [!Important]
> La clave de los Plugs es que se rijan por el contrato (implementación) del framework. De esta forma se pueden crear y usar sin mayor problema.

### Plugs Funciones

Tiene que hacer dos cosas:

- Aceptar dos parámetros: struct `Plug.Conn` y opciones
- Retornar un struct `Plug.Conn`

### Plugs Módulos

Tiene que hacer dos cosas:

- Definir una función `init/1` que inicia cualquier argumento u opciones para el plug
- Definir una función `call/2` que hace lo mismo que un plug tipo función:
	- Aceptar dos parámetros
	- Retornar `Plug.Conn`


# ¿Qué es un Repo?

 > (...) the repo (or repository) is a mapping to a data store. (...) normally the repo maps to an actual database.

The database is the component that actually stores the data; the repo is the application’s gateway to the data inside the database. Your repo will translate what you’d like from the database into “database speak” through a defined public interface.

# Umbrella Applications

> Although simple applications are typically structured in this way, you’ll be creating multiple applications—one for the business logic you’ve been working on and one for your Phoenix application. But you’ll also create another skeleton application to tie them together. This kind of pattern is called an umbrella application. The top-level umbrella application will contain the subapplications that contain the logic

Son una forma de agrupar un conjunto de aplicaciones que corresponden al mismo dominio. Por ejemplo, una Umbrella App podría estar compuesta de sub aplicaciones con diferentes responsabilidades:

- Una app tipo shell
- Una API / Web
- Un cliente que es una interfaz con otros sistema y se invoca en los proyectos anteriores

Así se ve crear una app umbrella:
```bash
$ mix new --umbrella auction_umbrella
* creating README.md
* creating .formatter.exs
* creating .gitignore
* creating mix.exs
* creating apps
* creating config
* creating config/config.exs

Your umbrella project was created successfully.
Inside your project, you will find an apps/ directory
where you can create and host many apps:

    cd auction_umbrella
    cd apps
    mix new my_app

Commands like "mix compile" and "mix test" when executed
in the umbrella project root will automatically run
for each application in the apps/ directory.
```

# Archivos .ex vs .exs

Los archivos .ex son para ser compilados.

Los archivos .exs son para ser interpretados.

# application en mix.exs

Este código:
```ruby
# Run "mix help compile.app" to learn about applications.
  def application do
    [
      extra_applications: [:logger],
      mod: {Auction.Application, []}
    ]
  end
```

> it’s what tells the compiler that you’d like to generate an .app file for your application. (...) "An .app file is a file containing Erlang terms that defines your application. Mix automatically generates this file based on your mix.exs configuration."

> An Elixir application compiles down to Erlang code, which will run on the BEAM virtual machine, and the application function provides additional instructions to the compiler about how to compile Erlang code.

# Comandos Mix

`mix compile`: para compilar un proyecto Mix.

`iex -S mix`: para usar en la consola de Elixir un proyecto Mix.

`mix hex.search [nombre_paquete]`: para buscar paquetes en el registro Hex.

`mix deps.get`: para instalar las dependencias listadas en el archivo `mix.exs`.

`mix deps.clean [nombre_paquete]`: para limpiar los rastros dejados por paquetes sacados del proyecto Mix.

# ¿Son las umbrella apps la forma de hacer proyectos en Elixir?

Siento un poco confusa y enredada la forma de configurar este proyecto de aprendizaje usando esta forma de Umbrella App. Crea las apps aquí, configura una cosa a nivel umbrella. ¿Por qué?

Parece que es la forma en que el autor prefiere las coas:

> This is where having strong boundaries between your interface (like a Phoenix web application) and your business logic has huge benefits.
> 
> As long as your business logic provides a public interface you can use to get the information you’re after, you can let it take care of where it keeps the data and how it gets it.
>
>  ==You want your web interface to be as naive as possible about the inner workings of the business logic==.

Tengo que averiguar esto.

Enlaces:

- [Why would you ever use an umbrella app instead of just a regular app?](https://www.reddit.com/r/elixir/comments/sjxyis/why_would_you_ever_use_an_umbrella_app_instead_of/)
- [Using an Elixir Umbrella](https://8thlight.com/insights/using-an-elixir-umbrella)

# Sigil `~p` en `redirect` en acciones de controlador

La acción `create` en el curso me tocó actualizar a hacer el redirect de esta forma. La cual es la forma moderna:
```erlang
{:ok, user} -> redirect(conn, to: ~p"/users/#{user}")
```

¿Qué hace `~p"/users/#{user}"`?

Los [docs](https://hexdocs.pm/phoenix/Phoenix.VerifiedRoutes.html#sigil_p/2) nos dicen que la función se llama `sigil_p/2`:
```
Generates the router path with route verification.

Interpolated named parameters are encoded via the [`Phoenix.Param`](https://hexdocs.pm/phoenix/Phoenix.Param.html) protocol.
```

Otro ejemplo en la vista edit de Item:
```erlang
<.link href={~p"/items/#{@item.data}"}> Back to item</.link>
```

## ¿Por qué usar el sigil `~p` en las rutas?

Pregunté a Claudio y dijo:

> 1. Verificación en tiempo de compilación - Si la ruta no existe, obtienes un error al compilar, no en runtime
> 2. Type safety - Verifica que los parámetros coincidan con lo definido en el router
> 3. Más simple - No necesitas recordar el nombre de la función helper (user_path, users_path, etc.)

Lo cual tiene bastante sentido para mí. Sobretodo la parte de tiempo de compilación.

# Vistas, Templates y Componentes

En un capítulo cree la vista `apps/auction_web/lib/auction_web/controllers/page_html.ex` también cree la carpeta `page_html` al mismo nivel junto con el template `apps/auction_web/lib/auction_web/controllers/page_html/home.html.heex`.

De todo esto me salen varias dudas.

## Módulo AuctionWeb.PageHTML (view)

El nombre del módulo es `AuctionWeb.PageHTML`. Esto es una view. Los [docs](https://hexdocs.pm/phoenix/request_lifecycle.html#a-new-controller) dicen:

> The modules responsible for rendering are called views. (...) Phoenix views are named after the controller and format (`HTML` in this case), so Phoenix is expecting a `HelloWeb.HelloHTML` module to exist (...)

Es por eso que que en mi caso la vista se llama `PageHTML`:

- Lleva el nombre del controlador `AuctionWeb.PageController`
- Y el formato que en este caso es `HTML`
	- Va en mayúsculas al parecer

## Templates

> [!Tip]
> Estos son los que en Rails llamamos vistas.

La vista/view es el módulo que se encargar de renderizar un template. El template es donde se pone el código HTML mezclado con Elixir. Veamos ambos a continuación.

**View - PageHTML**

```erlang
defmodule AuctionWeb.PageHTML do
  @moduledoc """
  This module contains pages rendered by PageController.

  See the `page_html` directory for all templates available.
  """
  use AuctionWeb, :html

  embed_templates "page_html/*"
end
```

**Template - home.html.heex**

```ruby
<.flash_group flash={@flash} />

<ul>
  <%= for item <- @items do %>
    <li>
      <strong>
        <%= item.title %>:
      </strong>

      <%= item.description %>
    </li>
  <% end %>
</ul>
```

En la view definimos qué templates renderizar con la función `embed_templates "page_html/*"`.

En el template ya vemos la mezcla de funciones (helpers) de Elixir y código HTML.

## Componentes

Cuando se generó el proyecto Phoenix con `mix phx.new.web auction_web --no-ecto` también se creó el archivo `apps/auction_web/lib/auction_web/components/core_components.ex`. Ahí encuentro varias cosas que he estado usando en el proyecto como:

- `flash_group`
	- En el template `home.html.heex` se usa `<.flash_group flash={@flash} />`.
- `simple_form`
	- En la vista `AuctionWeb.ItemHTML` se define una función que usa el componente `simple_form` para el formulario de crear/actualizar Item
- `button`
	- A su vez, el formulario de Item usa `button` para el botón submit del formulario.

El módulo `AuctionWeb.CoreComponents` define muchos componentes que tienen que ver con cosas de forms como `input`, `flash`, `label`. La ventaja de estos componentes es que sirven tal cual y componentes de la UI. Es que son eso. Son componentes para reusar en la UI de todo el proyecto.

Este es el código generado para el componente `button`.
```erlang
  @doc """
  Renders a button.

  ## Examples

      <.button>Send!</.button>
      <.button phx-click="go" class="ml-2">Send!</.button>
  """
  attr :type, :string, default: nil
  attr :class, :string, default: nil
  attr :rest, :global, include: ~w(disabled form name value)

  slot :inner_block, required: true

  def button(assigns) do
    ~H"""
    <button
      type={@type}
      class={[
        "phx-submit-loading:opacity-75 rounded-lg bg-zinc-900 hover:bg-zinc-700 py-2 px-3",
        "text-sm font-semibold leading-6 text-white active:text-white/80",
        @class
      ]}
      {@rest}
    >
      <%= render_slot(@inner_block) %>
    </button>
    """
  end
```

Aquí encuentro otra cosa que es el sigil `~H`. Los [docs lo explican](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#sigil_H/2).

### Sigil `~H`

Este es una función para escribir templates `HEEx` en código fuente. Los templates `HEEx` sirven para escribir HTML con Elixir embebido. Estos templates ofrecen:

- Built-in handling of HTML attributes
- An HTML-like notation for injecting function components
- Compile-time validation of the structure of the template
- The ability to minimize the amount of data sent over the wire
- Out-of-the-box code formatting via [`mix format`](https://hexdocs.pm/mix/Mix.Tasks.Format.html)

> [!Note]
> HEEX significa "HTML + EEx".

Todo esto es muy interesante y se ve cómo va un nivel más allá de las vistas de Rails. Por ejemplo, este error que me salió intentando correr el servidor de Phoenix:
```bash
== Compilation error in file lib/auction_web/views/user_html.ex ==
** (Phoenix.LiveView.Tokenizer.ParseError) lib/auction_web/views/user/show.html.heex:1:22: expected closing `>`
  |
1 | <h1>User details</h1.
  |                      ^
    (phoenix_live_view 1.0.0-rc.6) lib/phoenix_live_view/tokenizer.ex:719: Phoenix.LiveView.Tokenizer.raise_syntax_error!/3
    (phoenix_live_view 1.0.0-rc.6) lib/phoenix_live_view/tag_engine.ex:295: Phoenix.LiveView.TagEngine.handle_text/3
    (eex 1.15.4) lib/eex/compiler.ex:319: EEx.Compiler.generate_buffer/4
    (phoenix_live_view 1.0.0-rc.6) expanding macro: Phoenix.LiveView.HTMLEngine.compile/1
    lib/auction_web/views/user/show.html.heex: AuctionWeb.UserHTML.show/1
```

No había cerrado la etiqueta h1 correctamente y Phoenix explotó y además me indicó el error. Esto en Rails no pasaría a menos que fuera código ERB.

# `with` para pattern matching en secuencia

En el capítulo de configuración de la página de inicio de sesión se agrega esta función a `Auction`:
```erlang
def get_user_by_username_and_password(username, password) do
	with user when not is_nil(user) <- @repo.get_by(User, %{username: username}),
		true <- Password.verify_with_hash(password, user.hashed_password) do user
	else
		_ -> Password.dummy_verify
	end
end
```

Tiene bastante sintaxis poco convencional así que le pregunté a Claudio sobre todo.

## `with` para multiples operaciones

```erlang
with user when not is_nil(user) <- @repo.get_by(User, %{username: username}),
  true <- Password.verify_with_hash(password, user.hashed_password) do user
```

Lo primero de aquí es:
```erlang
user when not is_nil(user) <- @repo.get_by(User, %{username: username})
```

Hace varias cosas:

- Ejecuta la query `@repo.get_by(User, %{username: username})`
- Pattern matches el resultado a `user`
- El guard `when not is_nil(user)` sirve para garantizar que `user` existe
- Si devuelve nulo o falla algo, salta al `else`

Luego tenemos la segunda verificación de `with`:
```erlang
true <- Password.verify_with_hash(password, user.hashed_password)
```

- Solo se ejecuta si la primera es exitosa
- Espera que el resultado de `verify_with_hash` sea true
- Si devuelve nulo o falla algo, salta al `else`

Finalmente, está `do user`. Esto se ejecuta/devuelve si las operaciones son exitosas.

El else con `_ ->` captura cualquier cosa que llegue.

Claudio dice:
> Think of it as: "Try step 1, then step 2. If both work, return user. If either fails, run dummy verification."

## ¿Qué es `<-` y `->`?

Claudio dice que:
> <- (left arrow) - "draw from" or "bind from"

- Se usa en `with`, `for` y comprenhensions.
- Toma un valor de la derecha y lo matchea con lo de la izquierda

> [!Note]
> Example: user <- @repo.get_by(...) means "execute the function and bind the result to user"

Ahora la otra flecha:
> -> (right arrow) - "then" or "maps to"

- Se usa en funciones, `case`, `cond`, funciones anónimas
- Separa el patrón/condición del código a ejecutar

> [!Note]
> Example: _ -> Password.dummy_verify means "if anything matches _, then execute Password.dummy_verify"

En el contexto de la función de esta nota:
```erlang
with user <- @repo.get_by(...)  # <- binds result to user
	...
else
	_ -> Password.dummy_verify    # -> means "then execute this"
end
```

> [!Tip]
> Think of <- as assignment/binding and -> as consequence/execution.