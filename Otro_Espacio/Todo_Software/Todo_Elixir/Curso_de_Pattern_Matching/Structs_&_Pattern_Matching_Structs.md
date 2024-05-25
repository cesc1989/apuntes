# 🏙 Structs & Pattern Matching Structs
[Lección](https://thinkingelixir.com/course/pattern-matching/module-2/struct/).

A struct is an extension of a map with more strict rules about what keys it can have. A struct’s keys are atoms and cannot be strings.

A struct is defined *inside* a module and the name of the struct *is* the module itself.

    defmodule Player do
      defstruct [:username, :email, :score]
    end


## Valores por Defecto
    defmodule Player do
      defstruct username: nil, email: nil, score: 0
    end


## Un Struct es un Mapa
    gary = %Player{username: "Gary", score: 100}
    #=> %Player{email: nil, score: 100, username: "Gary"}
    
    is_map(gary)
    #=> true
    
    Map.get(gary, :score)      
    #=> 100
    
    gary.score
    #=> 100


## A struct can be like an OOP class

An Elixir struct is similar to a “class” in Object Oriented Programming languages.

The main difference for a struct is that the data structure and the functions are not explicitly *tied* together. We *define* them in the same place as a convenience both to the developer creating the data structure and writing the tests, but also to the developer using the struct.


# Pattern Matching Structs

[Lección](https://thinkingelixir.com/course/pattern-matching/module-3/pattern-matching-a-function-body-struct/).

If I perform the match as a map instead of the struct, I have no protections or guarantees. This alone is a significant benefit. It’s the difference of including the struct name or not.

    # This version is compile-time checked for Customer keys.
    def do_work(%Customer{name: name}) do
      # work
    end
    
    # This version cannot be checked.
    def do_work(%{name: name}) do
      # work
    end

When a struct is part of the Pattern Match, it is a guarantee that the data coming in *is* that struct. Not just a map with similar keys, but it *is that struct*. All the code inside that function clause can be written confidently knowing it can’t be something else with a similar structure.

It is totally valid to define a function clause that matches the struct just to have that assurance and protection.

    def do_work(%Customer{} = customer) do
      # work
    end

Inside this function clause I am guaranteed to have a Customer struct. This is opposed to a similar function declaration without it.

    def do_work(customer) do
      # work
    end


## Pruebas y Código de Pattern Matching de Structs

Los structs son:

    defmodule PatternMatching.Customer do
      defstruct [:name, :location, :contact_name, :contact_number, {:orders, []}, {:active, true}]
    
      @type t :: %Customer{
              name: String.t(),
              location: nil | String.t(),
              contact_name: nil | String.t(),
              contact_number: nil | String.t(),
              orders: list(),
              active: boolean()
            }
    end
    
    defmodule PatternMatching.User do
      defstruct [:name, :age, :hair, :gender, {:points, 0}, {:admin, false}, {:active, true}]
    
      @type t :: %User{
              name: String.t(),
              age: nil | integer(),
              hair: nil | String.t(),
              gender: nil | :male | :female,
              points: integer(),
              admin: boolean,
              active: boolean()
            }
    end

Pruebas y código que soluciona:

    describe "get_name/1" do
      test "returns a User or Customer name", %{customer: customer, user: user} do
        assert {:ok, "Kris Rowe"} == Structs.get_name(user)
        assert {:ok, "Widgets 4 Sale"} == Structs.get_name(customer)
      end
    
      test "returns error when given something that doesn't have a name" do
        assert {:error, "Doesn't have a name"} == Structs.get_name(123)
        assert {:error, "Doesn't have a name"} == Structs.get_name(%{stuff: true})
      end
    end
    
    def get_name(%User{name: name} = _user) do
      {:ok, name}
    end
    
    def get_name(%Customer{name: name} = _customer) do
      {:ok, name}
    end
    
    def get_name(_other) do
      {:error, "Doesn't have a name"}
    end

Otro

    describe "create_greeting/1" do
      test "creates a unique greeting for a Customer and User", %{customer: customer, user: user} do
        assert {:ok, "Greetings user Kris Rowe!"} == Structs.create_greeting(user)
        assert {:ok, "Howdy customer Widgets 4 Sale!"} == Structs.create_greeting(customer)
      end
    
      test "does not create a greeting when inactive", %{customer: customer, user: user} do
        # make the customer and user inactive
        customer = %Customer{customer | active: false}
        user = %User{user | active: false}
    
        assert {:error, "Recipient is inactive"} == Structs.create_greeting(customer)
        assert {:error, "Recipient is inactive"} == Structs.create_greeting(user)
      end
    end
    
    def create_greeting(%User{name: name, active: true} = _user) do
      {:ok, "Greetings user #{name}!"}
    end
    
    def create_greeting(%Customer{name: name, active: true} = _customer) do
      {:ok, "Howdy customer #{name}!"}
    end
    
    def create_greeting(%User{active: false} = _user) do
      {:error, "Recipient is inactive"}
    end
    
    def create_greeting(%Customer{active: false} = _customer) do
      {:error, "Recipient is inactive"}
    end

Finalmente:

    describe "deactivate_user/1" do
      test "updates user to set active to false", %{user: user} do
        {:ok, %User{} = updated} = Structs.deactivate_user(user)
        assert updated.active == false
      end
    
      test "returns an error if not a user", %{customer: customer} do
        assert {:error, "Not a User"} == Structs.deactivate_user(customer)
        assert {:error, "Not a User"} == Structs.deactivate_user(123)
      end
    end
    
    def deactivate_user(%User{} = user) do
      # {:ok, Map.put(user, :active, false)}
      # {:ok, Map.merge(user, %{active: false})}
      {:ok, %User{user | active: false}}
    end
    
    def deactivate_user(_other), do: {:error, "Not a User"}

