# 🤙🏽Enumerables y Funciones Anónimas - [Resistor Color]
Al resolver el ejercicio [*Resistor Color*](https://exercism.io/my/solutions/94ce97e86ba24873a2e50d923fbe7d93) me di cuenta que leí mejor el enunciado y aplique mejor el uso de funciones anónimas y una función de `Enum`.

Con estas pruebas:

    defmodule ResistorColorTest do
      use ExUnit.Case
    
      # @tag :pending
      test "returns black color code" do
        assert ResistorColor.code("black") == 0
      end
      # (...)
      # @tag :pending
      test "returns all colors" do
        colors = [
          "black",
          "brown",
          "red",
          "orange",
          "yellow",
          "green",
          "blue",
          "violet",
          "grey",
          "white"
        ]
    
        assert ResistorColor.colors() == colors
      end
    end

me di cuenta que el ejercicio se resolvía muy fácil usando la sentencia `case` y así lo hice y todas las pruebas pasaron. Sin embargo, me pareció muy sencillo. Leí de nuevo el enunciado y en esta oración me di cuenta que había que hacer otra cosa:


> Mnemonics map the colors to the numbers, that, when stored as an array, happen to map to their index in the array

Estaba claro que no tenía que usar un `case` sino valerme de una lista para definir los colores y los índices para determinar sus colores.

Así fue que llegué a la conclusión con:

    def code(color) do
      Enum.find_index(colors, fn(x) -> x == color end)
    end

Pero no fue sencillo de una vez. Resulta que función `[Enum.find_index/2](https://hexdocs.pm/elixir/1.10.4/Enum.html#find_index/2)` tiene aridad de dos parámetros. En la documentación de Elixir dice:

    find_index(enumerable, fun)


- Primer parámetro un enumerable
- Segundo parámetro una función

Aquí aprendí otra cosa. Es que a una función puedo pasarle otra función para que la ejecute por mí. Es decir, al usar `Enum.find_index` y pasarle la función anónima `fn(x) → x == color end` le estoy indicando a la primera que use la segunda para determinar el valor a hallar y retornar.

**Enlaces**

- [Sobre funciones anónimas](https://github.com/cesc1989/programming-elixir/blob/master/elixir-school/001-basics/006-functions.md#anonymous-functions)

