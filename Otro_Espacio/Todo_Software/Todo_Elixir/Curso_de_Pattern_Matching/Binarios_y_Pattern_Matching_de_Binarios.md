# Binarios y Pattern Matching de Binarios

The important thing to remember with binary pattern matching is where it works well and where it doesn’t. It works great in these situations:

- matching on a command-style prefix
- matching and unpacking a fixed size string (like for formatting)
- matching and unpacking data from a fixed size binary structure (like a header)

## A un prefijo de una cadena

A string is a binary type. We can match on the beginning of a string like this:
```erlang
defmodule StringTests do
	def match_greeting("Hello " <> subject), do: {:hello, subject}
	def match_greeting("Greetings " <> subject), do: {:greetings, subject}
	def match_greeting("Good morning!"), do: {:morning, nil}
	def match_greeting(_other), do: :unknown
end

StringTests.match_greeting("Hello Tom")                                 
#=> {:hello, "Tom"}

StringTests.match_greeting("Greetings Jane")         
#=> {:greetings, "Jane"}

StringTests.match_greeting("Good morning!")
#=> {:morning, nil}

StringTests.match_greeting("Buenos dias") 
#=> :unknown
```
