# 🥇Pattern Matching

Ejercicios en Replit.

```erlang
pattern = data
```

Three things that can *all* happen *at the same time* when a match is performed.

1. Match the **type** of data
2. Match the **shape** of data
3. **Bind values** to variables

# Imperativo vs Declarativo (pattern matching)

Tomemos de ejemplo este mapa que se ata a la variable `data`.

```erlang
data = %{name: "Howard", email: "howard@example.com"}
```

Ahora veamos esta función imperativa
```erlang
name =                               # result is bound to name
	if is_map(data) do                 # match type
		if Map.has_key?(data, :name) do  # match shape
			Map.get(data, :name)           # get :name value
		end
	end
	
name                      # name variable is bound to value
#=> "Howard"
```

Lo que hace la función es verificar que `data` es un Map, que tenga la llave `:name` y si la tiene asigno su valor a la variable `name`.

En palabras del autor del curso:

> I systematically “poke” the data blindly trying to feel out it’s shape. Once I know that it is a map and it has a `:name` key, then I can get that value and bind the `name` variable.

Con Pattern matching, todo esto se vuelve más sencillo en una sola expresión. En lugar de estar revisando la forma y los datos del mapa, con pattern matching es como decir “Este es el tipo y la forma de los datos que quiero, si los encuentras, atálos a esta variable”.

```erlang
%{name: name} = %{name: "Howard", email: "howard@example.com"}

name
#=> "Howard"
```

## Elegance in communication

The imperative version is a series of instructions that accomplishes something similar to the pattern matching version but it is much less clear. The imperative version tells us *how* to do it but a developer reading then code is left to figure out *what* is being done.

Pattern matching helps us create code that avoids nested **if** statements. Our code becomes flatter, clearer, easier to read and easier to maintain.

# Entiendo Pattern Matching usando case 

Veamos este código:
```erlang
data = %{name: "Howard", age: 35}

case data do
	%{name: "Howard"} -> "Yes sir Mr. Admin!"
	%{name: name} -> "Greetings #{name}!"
	%{age: age} -> "I don't know who you are, but you're #{inspect age} years old!"
	_other -> "Uhh.... what's that?"
end
```

> Tener en cuenta: el orden de los patrones importa. **El primer patrón que se encuentre o cause error, detendrá la ejecución de la sentencia.**
> 
> Se lo más especifico posible siempre de arriba hacia abajo.

Puede pasar tres cosas que procedo a explicar según tres casos.

    data = %{name: "Howard", age: 35}

Con esa forma, el patrón será encontrado. “Yes sir Mr. Admin”.

En esta que sigue

    data = %{name: "Mario" age: 35}

El segundo patrón será encontrado porque ocurre lo siguiente

    ${name: name} = ${name: "Mario"}

Se está atando el tipo y la forma a la variable `name` la cual es usada en el bloque de código luego de la flecha `→` en “Greeting #{name}!”

Por último, si ningún patrón se encuentra, se usará la última expresión. La última expresión que empiece con un guion bajo se deja como una forma de capturar todo lo que no sea capturado previamente.

Sino se especifica, ocurrirá una excepción cuando no se pueda encontrar ningún patrón.

    ** (CaseClauseError) no case clause matching: %{important_flag: true, level_1: %{level_2: %{more: "stuff", value: 123}, other: "stuff"}}


## Deeper Nested Data Matches

```erlang
deeper = %{
	important_flag: true,
	level_1: %{
		other: "stuff",
		level_2: %{
			value: 123,
			more: "stuff"
		}
	}
}
case deeper do
	%{important_flag: false} -> IO.inspect({:ok, 0})
	%{important_flag: true, level_1: %{level_2: %{value: value}}} -> IO.inspect({:ok, value})
	_other -> IO.inspect({:error, "invalido"})
end
```

De esa forma se puede sacar información anidada varios niveles solo con pattern matching sin usar `if`.


# En General

- The *pattern* goes on the left of the Match Operator (`=`). The data goes on the right.
- A Pattern Match can match the data’s *type*, *shape*, and *bind* variables to values all in a single statement.
- A Match Error occurs when no match could be made.
- **Pattern matching goes from top to bottom**. If the first pattern doesn’t match, the next pattern is checked and so on.
- The first pattern to match wins and takes the data.
- Make your top patterns more specific.
- The “`^`” Pin Operator lets you reference the *value* of a variable in a pattern.
- The “`_`” lets you define *shape* without binding to the value.
- A nested `if` statement is an anti-pattern.
- Lists are “linked lists” and it is cheaper to add to the front than it is to add to the end like an array.
- Lists are recursive, a list is made up of a “head” element and a tail that is itself a list.
- Strings can be pattern matched.
- Guard clauses are another level of a pattern match. They can be also be used to match *type* and *shape*.
- Guard clauses allow you to compare bound variables to each other in a pattern match.
- Pattern matching is most effective when you think differently about your code.

