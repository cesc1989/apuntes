# Tuplas

[Lección](https://thinkingelixir.com/course/pattern-matching/module-2/tuple/).

- A tuple’s elements are *ordered* and *fixed* in size.
- A tuple does **NOT** mean **TWO**. Tuples can have many elements. However, it is common to see a tuple with only two elements. 
- A common usage in Elixir is to return multiple result values from a function in a tuple.
- Tuples are best for a fixed number of elements. If you need a *dynamic* container that preserves order, use a `List` instead.

Another example of using a tuple to return multiple pieces of information at once might be the result of a function that splits a list of integers into odd and even sets.

```erlang
{odd_results, even_results} = split_odd_even([1, 2, 7, 12, 15])

odd_results
#=> [1, 7, 15]

even_results
#=> [2, 12]
```

