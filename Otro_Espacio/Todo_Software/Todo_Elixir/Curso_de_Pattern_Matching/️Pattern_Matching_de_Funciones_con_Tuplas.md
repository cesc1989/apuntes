# ⭐️Pattern Matching de Funciones con Tuplas
En la lección, hubo solo ejercicios de funciones que se resolvían con pattern matching de Tuplas.

A continuación, detallo las pruebas de cada función y su respectiva implementación.


    describe "day_from_date/1" do
      test "returns the day number from an erlang date" do
        assert 5 == Tuples.day_from_date({2018, 9, 5})
        assert 10 == Tuples.day_from_date({2018, 9, 10})
      end
    end
    
    def day_from_date({_year, _month, day}) do
      day
    end



    describe "has_three_elements?/1" do
      test "return true for 3 element tuples" do
        assert true == Tuples.has_three_elements?({:ok, 100, 200})
        assert true == Tuples.has_three_elements?({:ok, :dokey, :do})
        assert true == Tuples.has_three_elements?({10, 100, 1000})
        assert false == Tuples.has_three_elements?({10, 100})
        assert false == Tuples.has_three_elements?({10})
        assert false == Tuples.has_three_elements?({1, 2, 3, 4})
      end
    end
    
    def has_three_elements?({_, _, _}) do
      true
    end
    
    def has_three_elements?(_tuple) do
      false
    end



    describe "major_us_holiday/1" do
      test "return major US holiday for the month" do
        assert "Christmas" = Tuples.major_us_holiday({2018, 12, 4})
        assert "Christmas" = Tuples.major_us_holiday({2017, 12, 14})
        assert "Christmas" = Tuples.major_us_holiday({1984, 12, 24})
        assert "4th of July" = Tuples.major_us_holiday({2018, 7, 30})
        assert "4th of July" = Tuples.major_us_holiday({2020, 7, 3})
        assert "New Years" = Tuples.major_us_holiday({2018, 1, 24})
        assert "New Years" = Tuples.major_us_holiday({2019, 1, 20})
        assert "Uh..." = Tuples.major_us_holiday({2018, 2, 4})
        assert "Uh..." = Tuples.major_us_holiday({2018, 3, 4})
        assert "Uh..." = Tuples.major_us_holiday({2015, 4, 4})
        assert "Uh..." = Tuples.major_us_holiday({2030, 5, 4})
      end
    end
    
    def major_us_holiday({_year, 12, _day}), do: "Christmas"
    def major_us_holiday({_year, 7, _day}), do: "4th of July"
    def major_us_holiday({_year, 1, _day}), do: "New Years"
    
    def major_us_holiday(_erl_date) do
      "Uh..."
    end



    describe "greet_user/1" do
      test "greets when user was found" do
        assert "Hello Jim!" == Tuples.greet_user({:ok, "Jim"})
        assert "Hello Gwen!" == Tuples.greet_user({:ok, "Gwen"})
      end
    
      test "does not greet when there was an error" do
        assert "Cannot greet" == Tuples.greet_user({:error, "User not found"})
        assert "Cannot greet" == Tuples.greet_user({:error, "Failed to connect to DB"})
      end
    end
    
    def greet_user({:ok, username}), do: "Hello #{username}!"
    def greet_user({:error, _reason}), do: "Cannot greet"



    describe "add_to_result/1" do
      test "adds 10 to the result return in :ok tuple" do
        assert {:ok, 15} == Tuples.add_to_result({:ok, 5})
        assert {:ok, 10} == Tuples.add_to_result({:ok, 0})
        assert {:ok, 100} == Tuples.add_to_result({:ok, 90})
      end
    
      test "returns unmodified when not :ok" do
        expected = {:error, "It had problems"}
        assert ^expected = Tuples.add_to_result(expected)
    
        expected = {:error, 9}
        assert ^expected = Tuples.add_to_result(expected)
      end
    end
    
    def add_to_result({:ok, number}), do: {:ok, number + 10}
    def add_to_result(tuple), do: tuple

