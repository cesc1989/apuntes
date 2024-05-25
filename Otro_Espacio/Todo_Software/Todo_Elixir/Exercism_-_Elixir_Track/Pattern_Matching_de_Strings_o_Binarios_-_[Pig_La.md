# ✳️Pattern Matching de Strings o Binarios - [Pig Latin]
[Pig Latin](https://exercism.io/my/solutions/9b585d6bbe104cb98a9012152e5d15c2).

Me siento contento porque resolví el ejercicio, en la mayoría de las pruebas, usando Pattern Matching de binarios.

Así lo resolví al 90%

    defmodule PigLatin do
      @vowels ["a", "e", "i", "o", "u"]
      @consonants [
        "b",
        "c",
        "d",
        "f",
        "g",
        "h",
        "j",
        "k",
        "l",
        "m",
        "n",
        "p",
        "q",
        "r",
        "s",
        "t",
        "v",
        "w",
        "x",
        "y",
        "z"
      ]
      @double_sound ["ch", "qu", "th"]
      @triple_sound ["thr", "sch", "squ"]
      @y_or_x ["y", "x"]
    
      @doc """
      Given a `phrase`, translate it a word at a time to Pig Latin.
    
      Words beginning with consonants should have the consonant moved to the end of
      the word, followed by "ay".
    
      Words beginning with vowels (aeiou) should have "ay" added to the end of the
      word.
    
      Some groups of letters are treated like consonants, including "ch", "qu",
      "squ", "th", "thr", and "sch".
    
      Some groups are treated like vowels, including "yt" and "xr".
      """
      
      def translate(<<first :: binary-size(1)>> <> phrase) when first in @vowels do
        first <> phrase <> "ay"
      end
    
      def translate(<<f :: binary-size(1), s :: binary-size(1)>> <> phrase) when f in @y_or_x and s in @consonants do
        "#{f}#{s}#{phrase}ay"
      end
    
      def translate(<<first :: binary-size(3)>> <> phrase) when first in @triple_sound do
        phrase <> "#{first}ay"
      end
    
      def translate(<<first :: binary-size(2)>> <> phrase) when first in @double_sound do
        phrase <> "#{first}ay"
      end
    
      def translate(<<f :: binary-size(1), s :: binary-size(1), t :: binary-size(1)>> <> phrase) when f in @consonants and s in @consonants and t in @consonants do
        phrase <> "#{f}#{s}#{t}ay"
      end
    
      def translate(<<first :: binary-size(1), second :: binary-size(1)>> <> phrase) when first in @consonants and second in @consonants do
        phrase <> "#{first}#{second}ay"
      end
    
      def translate(<<first :: binary-size(1)>><> phrase) when first in @consonants do
        phrase <> "#{first}ay"
      end
    end
    

Para hacer pattern matching de strings se hace como en el curso indicaron.

Relacionado

- Sobre como hacer [Pattern Matching a un string](https://stackoverflow.com/questions/25896762/how-can-pattern-matching-be-done-on-text/25897316#25897316).
- Pattern matching de los [primeros N caracteres](https://stackoverflow.com/questions/40817583/is-there-a-way-to-match-the-first-n-characters-in-an-elixir-string) de un string..

