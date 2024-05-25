# 🎯Guard Clauses
You should note that the `is_integer(value)` guard clause includes the bound variable `value` from the argument. **A guard clause can be used on any bound variable to help define more about the pattern you want**.

A guard clause expression must evaluate to `true` or `false`. It is being used to determine if the function should be executed.


    def function_name(arg1) when guard_clause do
      # function body
    end
    
    defmodule Testing do
      def greet_integer(value) when is_integer(value), do: "Hello #{value}"
      def greet_integer(_other), do: "meh"
    end
    
    Testing.greet_integer(123)                          
    #=> "Hello 123"
    Testing.greet_integer("Jim")
    #=> "meh"

Remember there are 3 parts to pattern matching:

1. Match the data type
2. Match the data shape
3. Bind variables to values

Guard clauses can help us with parts 1 & 2.

# Guard Clauses Ayudan con el Tipo

Ejemplos:

    def return_numbers(value) when is_number(value), do: value
    def return_lists(value) when is_list(value), do: value
    def return_any_size_tuples(value) when is_tuple(value), do: value
    def return_maps(value) when is_map(value), do: value
    def run_function(fun) when is_function(fun), do: fun.()
# Guard Clauses Ayudan con la Forma
    defmodule TestingGuard do
      def place_order(%{status: status} = order) when status in ["pending", "cart"] do
        "Placing order for #{order.customer_id}!"
      end
      def place_order(_order) do
        "Not placing order"
      end
    end
    
    order_1 = %{status: "pending", total: 100, customer_id: 10}
    order_2 = %{status: "cancelled", total: 75, customer_id: 12}
    
    TestingGuard.place_order(order_1)
    #=> "Placing order for 10!"
    TestingGuard.place_order(order_2)
    #=> "Not placing order"


![](https://paper-attachments.dropbox.com/s_DC4D5F05441F6F01540C789A36CA520FB793484A68111075DD7AECAFCB31B93F_1606789900940_image.png)



- Guard clauses further define a pattern for data *type* and *shape*.
- Custom guard clauses make reusable business patterns easy to use.
- Guard clauses allow us to define a pattern that combines multiple function arguments.
## Más Ejemplos
      @adult_age 18
    
      def classify_user(%User{age: age} = _user) when is_nil(age) do
        {:error, "Age missing"}
      end
    
      def classify_user(%User{age: age} = _user) when age >= @adult_age do
        {:ok, :adult}
      end
    
      def classify_user(%User{age: age} = _user) when age >= 0 and age < @adult_age do
        {:ok, :minor}
      end
    
      def classify_user(%User{age: age} = _user) when age < 0 do
        {:error, "Age cannot be negative"}
      end
    
      def classify_user(_user) do
        {:error, "Not a user"}
      end
# Custom Guard Clauses
    # defguard clause_name(arg1) when guard_clause
    
    @adult_age 18
    
    defguard is_adult?(age) when age >= @adult_age
    defguard is_minor?(age) when age >= 0 and age < @adult_age
    
    def classify_user(%User{age: age} = _user) when is_adult?(age) do
      {:ok, :adult}
    end
    
    def classify_user(%User{age: age} = _user) when is_minor?(age) do
      {:ok, :minor}
    end

