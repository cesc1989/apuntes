# 🤘🏽Sobre Strings, Binaries y Charlists - [RNA Transcription]
Bueno, resulta que la primera vez que leo y estudio sobre Elixir se me dice que todos los strings son con comillas doble:

    "hola"

Resulta que me metí en Exercism para poder practicar con menos presión de la que tengo en Bowlarama Elixir y en [el segundo ejercicio](https://exercism.io/tracks/elixir/exercises/rna-transcription/solutions/fb86becd0cc4470abf101e98b7b429ae#solution-comment-168551) me tocó hacer trampa sin saber.

Para las pruebas estaban usando una cadena con comillas simple.

    defmodule RnaTranscriptionTest do
      use ExUnit.Case
    
      # @tag :pending
      test "transcribes guanine to cytosine" do
        assert RnaTranscription.to_rna('G') == 'C'
      end
      # (...)
      # @tag :pending
      test "it transcribes all dna nucleotides to rna equivalents" do
        assert RnaTranscription.to_rna('ACGTGGTCTTAA') == 'UGCACCAGAAUU'
      end
    end

En mí solución:

    defmodule RnaTranscription do
      @doc """
      Transcribes a character list representing DNA nucleotides to RNA
    
      ## Examples
    
      iex> RnaTranscription.to_rna('ACTG')
      'UGAC'
      """
      @spec to_rna([char]) :: [char]
      def to_rna(dna) do
              char_list = String.split(dna, "", trim: true)
    
              x = Enum.map(char_list, fn(c) ->
                      case c do
                              "G" -> "C"
                              "A" -> "U"
                              "C" -> "G"
                              "T" -> "A"
                      end
              end)
    
              Enum.join(x)
      end
    end

Me encontraba con el error:

    ** (FunctionClauseError) no function clause matching in String.split/3    
        
        The following arguments were given to String.split/3:
        
            # 1
            'ACTG'
        
            # 2
            ""
        
            # 3
            [trim: true]
        
        Attempted function clauses (showing 3 out of 3):
        
            def split(string, %Regex{} = pattern, options) when is_binary(string)
            def split(string, "", options) when is_binary(string)
            def split(string, pattern, options) when is_binary(string)
        
        (elixir 1.10.4) lib/string.ex:471: String.split/3

**¿Por qué? Resulta que las cadenas con comillas simples no son cadenas “normales”. Son algo llamado** ***character lists.***

Veamos → https://culttt.com/2016/03/21/working-strings-elixir/

## Strings como Binaries

En Elixir, los *String* son representados como *Binaries*, es decir, una secuencia de bytes.

    is_binary("Hola")
    true


## Strings como Character lists

En Elixir, un [*Character list*](https://hexdocs.pm/elixir/1.10.4/List.html#module-charlists) es una *String* envuelta en comillas simples.

    'hola'

A su vez no es un binario:

    is_binary('hola')
    false

¿Entonces que es un *character list?* Pues es una lista de enteros que representan a cada carácter de la cadena.

Si en `iex`, escribo la cadena como lista con los valores enteros:

    iex(12)> [104, 111, 108, 97]
    'hola'

Puedo saber la representación en entero de un carácter usando el símbolo de pregunta:

    iex(8)> ?h
    104
    iex(9)> ?o
    111
    iex(10)> ?l
    108
    iex(11)> ?a
    97

Las cadenas creadas con comillas dobles y simples no se pueden usar de manera intercambiada. Muchas funciones de Elixir verifican que el parámetro recibido sea del tipo esperado.

**¿Entonces, cuál es la gracia?**
En realidad, debo apuntar a usar las cadenas con comillas dobles de manera general cuando escribo código en Elixir.

Los *Character lists* existen porque así se representan en Erlang.

Se puede convertir de *String* a *Charlist* con dos funciones:

    String.to_charlist("hola")
    List.to_string('hola')


## Los Character List son Listas

Quiero decir que las puedo enumerar o usar el modulo `Enum` en ellas. Ejemplo de esto es [una de las soluciones](https://exercism.io/tracks/elixir/exercises/rna-transcription/solutions/e4cd184ac1c74020a117d65c3c41c890) en Exercism de este mismo ejercicio.

    defmodule RnaTranscription do
      @doc """
      Transcribes a character list representing DNA nucleotides to RNA
    
      ## Examples
    
      iex> RnaTranscription.to_rna('ACTG')
      'UGAC'
      """
      @spec to_rna([char]) :: [char]
      def to_rna(dna) do
         Enum.map(dna, 
          fn x -> 
            case x do
              ?A -> ?U
              ?C -> ?G
              ?T -> ?A
              ?G -> ?C
              _ -> "Error"
            end
          end)
      end
    end

En la solución vemos como se itera la lista de caracteres, se compara cada uno usando el valor entero y al final se está generando un arreglo con la conversión.

