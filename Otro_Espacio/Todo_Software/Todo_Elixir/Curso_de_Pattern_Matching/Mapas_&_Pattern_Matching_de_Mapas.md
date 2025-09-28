# 🏞 Mapas & Pattern Matching de Mapas

[Lección](https://thinkingelixir.com/course/pattern-matching/module-2/map/).

## Rebinding

In Elixir, we don’t call it “variable assignment” because we aren’t “assigning” a value to a variable. **We are “binding” a variable to a value**.

Elixir allows us to **re-bind** a variable. Erlang and other functional programming languages don’t even allow that. Instead, I’d have to create a new variable for every change. *Elixir makes this easier and it can deceptively feel like how it works in other languages.*

## Pattern Matching Maps

[Lección](https://thinkingelixir.com/course/pattern-matching/module-3/pattern-matching-a-function-body-maps/).

Maps are your go-to key-value collections. You are likely to use them a lot. When building a web application, the request data submitted by a client is represented as a map with string keys.

There is a common code pattern we use in Elixir that is worth exploring.

```erlang
web_params = %{"name" => "John", "email" => "john@example.com"}

defmodule Testing do
	def do_work(%{"email" => email} = params) do
		IO.inspect email
		IO.inspect params
		"Sent an email to #{email} addressed to #{params["name"]}"
	end
end

Testing.do_work(web_params)
#=> "john@example.com"
#=> %{"email" => "john@example.com", "name" => "John"}
#=> "Sent an email to john@example.com addressed to John"
```

What’s special here is that we are pattern matching for some data we care about *and* still binding everything passed in as `params`.

This is a common code pattern. You can use this same approach everywhere you pattern match.

```erlang
%{"email" => email} = params = %{"name" => "John", "email" => "john@example.com"}

email
#=> "john@example.com"
params
#=> %{"email" => "john@example.com", "name" => "John"}
```

I want to Pattern Match and access a portion of the data when it is passed in, but I still want the full `customer` data because I pass it on to another function called `notify_customer`.

```erlang
defmodule Billing do
	def apply_charge(%{id: customer_id} = customer, charge) do
		record_charge(customer_id, charge)
		notify_customer(customer, charge)
	end
end
```

## Binding Nested Maps

They also elegantly handle nested data. When pattern matching, it is important to understand that you can also easily bind to a *nested* map. This can be very handy when working with web requests, and actually gets used frequently.

```erlang
params = %{
	"customer" => %{
		"id" => 123,
		"name" => "Willy Wonka Chocolates",
		"bonuses" => %{
			"employees" => %{
				"Oompa 1" => 1_000,
				"Oompa 2" => 2_000,
				"Hillary" => 1_500,
				"Oompa 3" => 500
			},
			"total" => 5_000
		}
	}
}

defmodule NestedBinding do

	def award_bonuses(%{"customer" => %{"bonuses" => %{"total" => bonus_total} = bonuses}} = _params) do
		IO.inspect bonus_total, label: "TOTAL TO VALIDATE"
		IO.inspect bonuses, label: "BONUSES"
		# TODO: validate intended total and employee amounts
		:ok
	end
end

NestedBinding.award_bonuses(params)
#=> TOTAL TO VALIDATE: 5000
#=> BONUSES: %{
#=>   "employees" => %{
#=>     "Hillary" => 1500,
#=>     "Oompa 1" => 1000,
#=>     "Oompa 2" => 2000,
#=>     "Oompa 3" => 500
#=>   },
#=>   "total" => 5000
#=> }
#=> :ok
```

Notese el pattern matching en la definición de la función:
```erlang
%{"customer" => %{"bonuses" => %{"total" => bonus_total} = bonuses}} = _params
```

Se está emparejando todo el cuerpo de la petición a la variable `_params`. Asímismo, se empareja la llave `"``bonuses"` al variable `bonuses` y de esta misma variable se empareja la llave `"``total``"` para emparejarla a la variable `bonus_total`.

# Ejercicio de Pattern Matching de Mapas

Casos de pruebas:
```erlang
defmodule PatternMatching.MapsTest do
	describe "classify_response/1" do
		setup _ do
			success = %{"success" => true, "token" => "syriuC2ia",
									"account" => %{"status_code" => "1000"},
									"messages" => %{"general" => %{"result_code" => 0}}}
			throttle = %{"success" => false, "token" => nil,
									"account" => nil,
									"messages" => %{"general" => %{"result_code" => 3}}}
			frozen = %{"success" => false, "token" => "syriuC2ia",
									"account" => %{"status_code" => "3001"},
									"messages" => %{"general" => %{"result_code" => 0}}}
			invalid = %{"success" => false, "token" => "invalid",
									"account" => %{"status_code" => ""},
									"messages" => %{"general" => %{"result_code" => -1}}}

			{:ok, %{invalid: invalid, throttle: throttle, frozen: frozen, success: success}}
		end

		test "returns token value when valid", %{success: success} do
			# successful when success is "true", return the token value
			assert {:ok, "syriuC2ia"} == Maps.classify_response(success)
		end

		test "returns error when invalid", %{invalid: invalid} do
			# invalid response have a success "false" and a result_code of "-1"
			assert {:error, :invalid} == Maps.classify_response(invalid)
		end

		test "returns retry when throttled", %{throttle: throttle} do
			# throttled response have a success "false" and a result_code of "3"
			assert {:error, :retry} == Maps.classify_response(throttle)
		end

		test "returns error with reason when frozen", %{frozen: frozen} do
			# frozen response have a success "false" and a status_code of "3001"
			assert {:error, :frozen} == Maps.classify_response(frozen)
		end
	end
end
```

Solución en código:
```erlang
defmodule PatternMatching.Maps do
	def classify_response(%{"success" => true, "token" => token} = _response) do
		{:ok, token}
	end

	def classify_response(%{"success" => false, "messages" => %{"general" => %{"result_code" => 3}}} = _response) do
		{:error, :retry}
	end

	def classify_response(%{"success" => false, "account" => %{"status_code" => "3001"}} = _response) do
		{:error, :frozen}
	end

	def classify_response(%{"success" => false, "messages" => %{"general" => %{"result_code" => -1}}} = _response) do
		{:error, :invalid}
	end
end
```

En vez de usar bloques de código condicionales, con pattern matching se empareja la respuesta con el tipo y forma de datos que necesitamos para las variantes de cada función.

