# 🥐 Listas & Pattern Matching de Listas

[Lección](https://thinkingelixir.com/course/pattern-matching/module-2/list/).

```erlang
my_list = [1, 2, 3, 4, 5]
```

This looks like an Array in other languages. However, **a List doesn’t** ***act*** **like an Array**.

A list in Elixir (and other functional programming languages) is actually a recursive data structure. Internally implemented as a linked list.

When you realize an Elixir list is an immutable linked list, it’s behavior makes sense and how you use it becomes clear.

![[elixir.list.png]]

## Variables son apuntadores y no asignaciones

`my_list` is “bound” to the list. The variable isn’t “assigned” the value of the list, **it is said to be “bound”. Variables in Elixir are all “bound” to their values.** You can think of “bound” as meaning “is pointing to”

Adding to the *front* of the List, or the “head” is very cheap.

# Pattern Matching de Listas

```erlang
[a, b, c | rest] = [1, 2, 3, 4, 5]

a
#=> 1
b
#=> 2
c
#=> 3
rest
#=> [4, 5]
```

El emparejamiento anterior se puede ver así:

- Con una lista de al menos 3 elementos
- ata el 1ero a la variable a
- el 2do a la variable b
- el 3ero a la variable c
- lo sobrante, átalo a la variable rest

## Ejercicios con pruebas y código

```erlang
    describe "is_empty?/1" do
      test "match an empty list" do
        assert true == Lists.is_empty?([])
        assert false == Lists.is_empty?(["hello"])
        assert false == Lists.is_empty?([1, 2, 3])
    
        assert false == Lists.is_empty?("not even a list!")
      end
    end
    
    def is_empty?([]), do: true
    def is_empty?(_list), do: false


    describe "has_1_item?/1" do
      test "match a list with exactly 1 item" do
        assert true == Lists.has_1_item?([1])
        assert false == Lists.has_1_item?([])
        assert false == Lists.has_1_item?([1, 2, 3])
        assert true == Lists.has_1_item?(["one thing"])
      end
    end
    
    def has_1_item?([_]), do: true
    def has_1_item?(_rest), do: false


    describe "at_least_one?/1" do
      test "match a list with at least 1 item" do
        assert true == Lists.at_least_one?([1])
        assert false == Lists.at_least_one?([])
        assert true == Lists.at_least_one?([1, 2, 3])
        assert true == Lists.at_least_one?(["one thing"])
        assert true == Lists.at_least_one?([3, 4, 5, 6, 7, 8])
      end
    end
    
    def at_least_one?([_ | _rest]), do: true
    def at_least_one?([]), do: false


    describe "return_first_item/1" do
      test "return the 1st item in list" do
        assert 1 == Lists.return_first_item([1])
        assert :error == Lists.return_first_item([])
        assert 10 == Lists.return_first_item([10, 20, 30])
      end
    end
    
    def return_first_item([head | _rest]), do: head
    def return_first_item([]), do: :error


    describe "starts_with_1?/1" do
      test "match lists that start with 1" do
        assert true == Lists.starts_with_1?([1, 2, 3])
        assert false == Lists.starts_with_1?([0, 1, 2])
        assert true == Lists.starts_with_1?([1])
        assert false == Lists.starts_with_1?([])
        assert true == Lists.starts_with_1?([1, 100, 1000])
      end
    end
    
    def starts_with_1?([1 | _rest]), do: true
    def starts_with_1?(_list), do: false


    describe "sum_pair/1" do
      test "sums items in list when exactly 2 items" do
        assert 3 == Lists.sum_pair([1, 2])
        assert 10 == Lists.sum_pair([5, 5])
        assert :error == Lists.sum_pair([])
        assert :error == Lists.sum_pair([1])
        assert :error == Lists.sum_pair([1, 2, 3])
      end
    end
    
    def sum_pair([first, second]), do: first + second
    def sum_pair(_list), do: :error


    describe "sum_first_2/1" do
      test "replace the first 2 items of a list with their sum" do
        assert [9, 3, 2, 1] == Lists.sum_first_2([5, 4, 3, 2, 1])
        assert [3, 3] == Lists.sum_first_2([1, 2, 3])
        assert [1] == Lists.sum_first_2([0, 1])
    
        # returns list if fewer than 2
        assert [1] == Lists.sum_first_2([1])
        assert [] == Lists.sum_first_2([])
      end
    end
    
    def sum_first_2([first, second | rest]) do
      [first + second | rest]
    end
    
    def sum_first_2(list), do: list
```

