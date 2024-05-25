# 🕶Iteración de colecciones y las Comprehensions - [Strain]
Me tomó una hora darme cuenta que no iba a poder solucionar el ejercicio llamado [Strain](https://exercism.io/my/solutions/f3ad61c3ac17431f9b053d1621d7df6c). ¿Qué pasó? Entendí mal todo el enunciado y tampoco interpreté bien el código de las pruebas para tener una idea de qué debía hacer.

Cuando leí las soluciones de otras personas de Exercism, me di cuenta de mi error y aprendí algunas cosas.

## Las listas o colecciones hay que iterarlas

Con esto me refiero a todo lo que sea una lista o una colección, hay que recorrerla con el modulo `Enum`.

Puede que *pattern matching* y recursividad vengan al caso pero lo primero es iterar.

Si vemos una de las pruebas del ejercicio:

    test "keep everything" do
      assert Strain.keep([1, 2, 3], fn e -> e < 10 end) == [1, 2, 3]
    end

y me fijo en la función anónima que se envía como segundo argumento:

    fn e -> e < 10 end)

se logra ver que está ejecutando una acción determinada sobre un elemento determinado.

Quiere decir que la función hará algo sobre cada elemento de la lista y para eso ¡tenía que iterarla!

¿Cuál era la implementación?

    def keep(list, fun) do
      for x <- list, fun.(x), do: x
    end

Otras soluciones usaban `Enum.reduce`.

## Comprehensions

En resumen, son azúcar sintáctica sobre funciones de iteración de listas.


- `for` → https://hexdocs.pm/elixir/Kernel.SpecialForms.html#for/1
- [Elixir School Comprenhesions](https://elixirschool.com/en/lessons/basics/comprehensions/)
- [Elixir docs Comprehensions](https://elixir-lang.org/getting-started/comprehensions.html)

La línea

    for x <- list, fun.(x), do: x

es un uso de una comprensión. Permite resolver el problema de una forma más corta que usando `Enum.reduce` o `Enum.map`.

