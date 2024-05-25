# ❌Sobre ejercicio Grep - [Grep]
Otro ejercicio más donde no logro completar. Ni siquiera pude resolver el primer test.

Hice el intento, leí, investigué y probé. Me desanimó aún más cuando veo las respuestas de la comunidad y son algoritmos que lucen, en cierto modo, avanzados.

Por ejemplo este de “[the pirate monkey](https://exercism.io/tracks/elixir/exercises/grep/solutions/ba1675f6fa504bdea038a66ee0e2bb84)”. Analicemos:

    defmodule Grep do
      @spec grep(String.t(), [String.t()], [String.t()]) :: String.t()
      def grep(pattern, [], [file]) do
        File.stream!(file)
        |> Stream.filter(&String.contains?(&1, pattern))
        |> Enum.join()
      end
    
      def grep(pattern, ["-i"], [file]) do
        File.stream!(file)
        |> Stream.filter(
          &String.contains?(
            String.downcase(&1),
            String.downcase(pattern)
          )
        )
        |> Enum.join()
      end
    
      def grep(pattern, ["-l"], [file]) do
        stream = File.stream!(file)
    
        if Enum.any?(stream, &String.contains?(&1, pattern)) do
          file <> "\n"
        else
          ""
        end
      end
    
      def grep(pattern, ["-n"], [file]) do
        File.stream!(file)
        |> Stream.with_index()
        |> Stream.filter(fn {line, _} -> String.contains?(line, pattern) end)
        |> Stream.map(fn {line, i} -> "#{i + 1}:#{line}" end)
        |> Enum.join()
      end
    
      def grep(pattern, ["-v"], [file]) do
        File.stream!(file)
        |> Stream.reject(&String.contains?(&1, pattern))
        |> Enum.join()
      end
    
      def grep(pattern, flags, [file]) do
        flags = Enum.sort(flags)
        stream = File.stream!(file)
    
        case Enum.sort(flags) do
          ["-i", "-n", "-x"] ->
            stream
            |> Stream.with_index()
            |> Stream.filter(fn {line, _} ->
              String.downcase(pattern) <> "\n" == String.downcase(line)
            end)
            |> Stream.map(fn {line, i} -> "#{i + 1}:#{line}" end)
    
          ["-x"] ->
            Stream.filter(stream, &(&1 == pattern <> "\n"))
    
          _ ->
            []
        end
        |> Enum.join()
      end
    
      def grep(pattern, flags, files) do
        grep(pattern, flags, files, "")
      end
    
      defp grep(_, _, [], acc), do: acc
    
      defp grep(pattern, flags, [h | t], acc) do
        unless "-l" in flags do
          result =
            grep(pattern, flags, [h])
            |> String.split("\n", trim: true)
            |> Stream.map(&"#{h}:#{&1}\n")
            |> Enum.join()
    
          grep(pattern, flags, t, acc <> result)
        else
          grep(pattern, flags, t, acc <> grep(pattern, ["-l"], [h]))
        end
      end
    end

Donde las condiciones de cada “flag” pedida en el ejercicio las maneja usando la sobre carga de funciones y así crea la misma función con diferente aridad.

    def grep(pattern, [], [file]) do end
    
    def grep(pattern, ["-i"], [file]) do end
    
    def grep(pattern, ["-n"], [file]) do end

Usa `Stream.filter!` en vez de `File.read` (como estaba intentando yo) y la cadena final la retorna concatenando (con `<>`)

