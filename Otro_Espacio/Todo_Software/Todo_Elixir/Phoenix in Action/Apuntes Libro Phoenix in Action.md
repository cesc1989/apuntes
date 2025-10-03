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

No me dice mucho eso por ahora.

# ¿Qué es un Repo?

 (...) the repo (or repository) is a mapping to a data store. (...) normally the repo maps to an actual database.

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