# 🎯Sobre el ejercicio Roman Numerals - [Roman Numerals]
Así lo resolví:

    defmodule RomanNumerals do
      @naturals [1000, 900, 500, 400, 100, 50, 40, 10, 9, 5, 4, 1]
      @romans %{
        1=>"I",
        4=>"IV",
        5=>"V",
        9=>"IX",
        10=>"X",
        40=>"XL",
        50=>"L",
        90=>"XC",
        100=>"C",
        400=>"CD",
        500=>"D",
        900=>"CM",
        1000=>"M"
      }
    
      @doc """
      Convert the number to a roman number.
      """
    
      @spec numeral(pos_integer) :: String.t()
      def numeral(0), do: ""
      
      def numeral(1), do: "I"
    
      def numeral(3000), do: "MMM"
    
      def numeral(number) do
        separator = Enum.find(@naturals, fn(x) -> x <= number end)
    
        {_nat, roman_str} = Enum.find(@romans, fn({k,_v}) -> separator == k end)
    
        roman_str <> numeral(number - separator)
      end
    end
    


## Enlaces de ayuda
- Aquí dicen la estrategia para [convertir de naturales a romanos](https://www.rapidtables.com/convert/number/how-number-to-roman-numerals.html).
- De este [gist](https://gist.github.com/mecampbellsoup/7001539) tome ideas.
- Código [Ruby que hace](https://github.com/nadavmatalon/roman_converter/blob/master/lib/roman_converter.rb) la conversión.
- [Cómo iterar un Map en Elixir](https://stackoverflow.com/questions/39937948/loop-through-a-maps-key-value-pairs).
- Loops en lenguajes tradicionales y su [versión en Elixir.](https://inquisitivedeveloper.com/lwm-elixir-73/)

