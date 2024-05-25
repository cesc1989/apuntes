# ✌🏽Sobre funciones de String y Enum.frequencies - [Word Count]
Resolviendo mi [tercer ejercicio en Exercism](https://exercism.io/my/solutions/6d0b5aaf61db4d729deb6d17fc181314), aprendí varias cosas en la documentación del módulo String y Enum.

Así lo resolví:

    defmodule WordCount do
      @doc """
      Count the number of words in the sentence.
    
      Words are compared case-insensitively.
      """
      @spec count(String.t()) :: map
      def count(sentence) do
              String.downcase(sentence) |>
              String.split([" ", ":", "!!&@$%^&", ",", "_"], trim: true) |>
              Enum.frequencies()
      end
    end


## String.split/3

Llama mí atención que a `String.split/3`, en el segundo parámetro que es el patrón para dividir la cadena, admite una lista de cadenas para aplicar el proceso.

Había intentado remover los caracteres con `String.trim`:

    String.trim("javascript!!&@$%^&", "!!")
    "javascript!!&@$%^&"

pero no obtenía ningún resultado. La clave está en que dicha función necesita el patrón o secuencia exacta para que pueda ser removida.

    String.trim("javascript!!&@$%^&", "!!&@$%^&")
    "javascript"
## Enum.frequencies/1

Esta [función](https://hexdocs.pm/elixir/1.10.4/Enum.html#frequencies/1) me sorprendió mucho y me recordó mucho a las librerías de Ruby que dan muchas funciones para problemas básicos.

Lo que hace es retornar un mapa con llaves únicas de cada elemento de un enumerable y los valores como la cantidad de veces que aparece la llave en el enumerable.

    Enum.frequencies(["ant", "buffalo", "ant", "ant", "buffalo", "dingo"])
    %{"ant" => 3, "buffalo" => 2, "dingo" => 1}

