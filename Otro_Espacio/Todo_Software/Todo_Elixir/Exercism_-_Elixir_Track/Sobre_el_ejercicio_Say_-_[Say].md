# ❌Sobre el ejercicio Say - [Say]
El ejercicio tiene la etiqueta “Pattern matching” pero muchas de las soluciones de la comunidad, me parece que distan de usarlo o lo usan de tal forma que me parece absurdo y no hice el ejercicio.

Al comienzo me interesaba un poco y tenía una leve idea de cómo empezar pero cuando decidí ver respuestas enviadas y ver gente usando un montón de métodos para casos puntuales y hasta la sentencia `case`, decidí saltar este ejercicio.

Creo que me está pasando que tengo una muy alta expectativa de Elixir y pienso que algunas cosas pasaran por arte de magia **y no será así**.

También siento que Exercism es un lugar excesivo con respecto al código que se sube o sobre cómo resolver algo **y en realidad tampoco es así**.

**No estoy atado a nada. Debo resolver los ejercicios y disfrutar y aprender. Más nada.**

Veamos este donde se usa Pattern Matching:

    defmodule Say do
      @hundred 100
      @thousand 1_000
      @million @thousand * @thousand
      @billion @million * @thousand
      @trillion @billion * @thousand
    
      @range 0..(@trillion - 1)
    
      @small %{
        10 => "ten",
        1 => "one",
        11 => "eleven",
        2 => "two",
        12 => "twelve",
        20 => "twenty",
        3 => "three",
        13 => "thirteen",
        30 => "thirty",
        4 => "four",
        14 => "fourteen",
        40 => "forty",
        5 => "five",
        15 => "fifteen",
        50 => "fifty",
        6 => "six",
        16 => "sixteen",
        60 => "sixty",
        7 => "seven",
        17 => "seventeen",
        70 => "seventy",
        8 => "eight",
        18 => "eighteen",
        80 => "eighty",
        9 => "nine",
        19 => "nineteen",
        90 => "ninety"
      }
    
      @doc """
      Translate a positive integer into English.
      """
      @spec in_english(integer) :: {atom, String.t()}
      def in_english(number) when number not in @range, do: {:error, "number is out of range"}
    
      def in_english(0), do: {:ok, "zero"}
    
      def in_english(number), do: {:ok, words(number)}
    
      defp words(number) when number <= 20 do
        @small[number]
      end
    
      defp words(number) when number < @hundred do
        case rem(number, 10) do
          0 -> @small[number]
          rem -> @small[number - rem] <> "-" <> @small[rem]
        end
      end
    
      defp words(number) when number < @thousand do
        big(number, @hundred, "hundred")
      end
    
      defp words(number) when number < @million do
        big(number, @thousand, "thousand")
      end
    
      defp words(number) when number < @billion do
        big(number, @million, "million")
      end
    
      defp words(number) do
        big(number, @billion, "billion")
      end
    
      defp big(number, base, suffix) do
        units = div(number, base)
        words = words(units) <> " " <> suffix
    
        case rem(number, base) do
          0 -> words
          rem -> words <> " " <> words(rem)
        end
      end
    end

Este tiene la peculiaridad que usa [atributos de módulo](https://elixir-lang.org/getting-started/module-attributes.html) para almacenar valores globales.

También veamos las clausulas vigías(guards que también son *pattern matching*) que usó:

    # El parametro se detecta. No es necesario hacer number == 0
    def in_english(0), do: {:ok, "zero"}
    
    # Acá usó dos macros vigia "not" e "in"
    def in_english(number) when number not in @range, do: {:error, "number is out of range"}
    
    # Acá otra guard con una condición y luego un case
    defp words(number) when number < @hundred do
      case rem(number, 10) do
        0 -> @small[number]
        rem -> @small[number - rem] <> "-" <> @small[rem]
      end
    end
    
    defp words(number) when number < @thousand do
      big(number, @hundred, "hundred")
    end

Ver sobre los [Guards y sus macros por defecto:](https://hexdocs.pm/elixir/1.10.4/Kernel.html#module-guards)

- `in/2` → https://hexdocs.pm/elixir/1.10.4/Kernel.html#in/2
- `not/1` → https://hexdocs.pm/elixir/1.10.4/Kernel.html#not/1



