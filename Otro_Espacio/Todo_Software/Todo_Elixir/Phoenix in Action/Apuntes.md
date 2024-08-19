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