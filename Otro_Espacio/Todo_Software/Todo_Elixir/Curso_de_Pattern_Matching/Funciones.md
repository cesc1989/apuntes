# Funciones

[Lección](https://thinkingelixir.com/course/pattern-matching/module-2/modules-and-functions/).

## Function Return Values

There is no way to *not* return a value. Given that this is Functional Programming, **every function returns a value! You may choose to ignore it, but it will return** ***something*****.**

## No Early Return

The short answer is, “Erlang doesn’t have explicit returns either and Elixir is built on Erlang.”

## ¿Si no quiero retornar nada?

There are times when a function does some work or creates a side-effect and there is *nothing* meaningful to return. A common pattern you’ll see in Elixir and Erlang is that those functions return the atom `:ok`.

```erlang
defmodule Testing do
	def do_stuff do
		# do stuff that can't fail or any errors are handled
		:ok
	end
end

Testing.do_stuff
#=> :ok
```
