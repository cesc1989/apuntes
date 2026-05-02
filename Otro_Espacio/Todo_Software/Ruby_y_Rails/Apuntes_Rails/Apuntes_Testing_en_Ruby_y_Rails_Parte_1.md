# Apuntes Testing en Ruby y Rails. Parte 1

- `ActionDispatch::Http::Parameters::ParseError: 822: unexpected token at`: [Stack Overflow](https://stackoverflow.com/questions/23570953/how-can-rspec-test-http-calls-with-body-and-headers) - [RSpec Relish](https://relishapp.com/rspec/rspec-rails/docs/request-specs/request-spec#providing-json-data)
- About `assign` in tests for instance variables: [Medium](https://medium.com/the-code-review/assigns-in-ruby-on-rails-how-you-can-test-instance-variables-in-views-and-controllers-13450a6b2472) - [SO](https://stackoverflow.com/a/21080501/1407371) - [rspec-rails](https://github.com/rspec/rspec-rails#assignkey-val)
- `respond_to` matcher, no presente en los docs de `rspec-expectations`: [reslishapp](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/respond-to-matcher) - [SO](https://stackoverflow.com/a/9120009/1407371)

# Sobre las gemas byebug, pry y pry-byebug

Escribir sobre sus diferencias y cómo usarlas.
- pry byebug https://github.com/deivid-rodriguez/pry-byebug
- byebug https://github.com/deivid-rodriguez/byebug

Luego de la reunión con Garyn:
- Byebug y ruby/debug -> debuggers
- Pry -> un IRB/rails console más poderoso

# Testing de Active Record Scopes

- [Stack Overflow](https://stackoverflow.com/questions/6485379/rails-testing-named-scopes-with-rspec)
- [Simone Carletti](https://simonecarletti.com/blog/2009/06/how-to-test-rails-activerecord-named-scopes/)
- [RSpec Docs](https://relishapp.com/rspec/rspec-mocks/v/3-6/docs/basics/scope)


# Cómo Testear Classes y/o Módulos

- Test class methods with RSpec: [SO, first](https://stackoverflow.com/questions/8004591/how-to-test-class-methods-in-rspec) - [then a gist](https://gist.github.com/knzconnor/816049) in which a comment points to use `allow`
- Test class methods with `described_class` instead of a mock. [Stack Overflow](https://stackoverflow.com/a/29128669/1407371)
- Test modules in rspec: [SO](http://stackoverflow.com/questions/1542945/testing-modules-in-rspec)

# Rubocop RSpec y MultipleMemoizedHelpers

Estaba terminando una característica en Patient Forms. Había actualizado Rubocop e instalado las gemas que acompañan: rubocop-rails y rubocop-rspec. Cuando intento hacer el commit, el pre-commit se queja con:
```
RSpec/MultipleMemoizedHelpers: Example group has too many memoized helpers [6/5]
```

Lo cual no entendía hasta que encontré el issue que trata al respecto.

Todo el fondo es por el artículo de Thoughtbot donde dicen que es mejor no usar `let` sino variables locales o métodos para armar los tests.

Ellos apelan que se procede a caer en el smell Mystery Guest ya que los *examples* no se pueden revisar sin el contexto y su configuración previa.

Enlaces:

- [Docs del Cop](https://docs.rubocop.org/rubocop-rspec/cops_rspec.html#rspecmultiplememoizedhelpers)
- [Issue al respecto](https://github.com/rubocop/rubocop-rspec/issues/862)
- [Artículo de Thoughtbot](https://thoughtbot.com/blog/lets-not)
- [Discusión más completa al respecto](https://github.com/rubocop/rspec-style-guide/issues/94)

# RSpec pre-commit hooks

- RSpec Git Pre-commit Hook → https://github.com/jags84/RSpec-Pre-commit-Git-Hook
- Tips for using a git pre-commit hook → https://codeinthehole.com/tips/tips-for-using-a-git-pre-commit-hook/

# How to test ActiveRecord callbacks

[Stack Overflow](https://stackoverflow.com/questions/16677718/how-to-test-models-callback-method-independently).

Callback and Callback behavior are independent tests. If you want to check an `after_save` callback, you need to think of it as two things:

1. Is the callback being fired for the right events?
2. Is the called function doing the right thing?

Para el comportamiento (behavior):
```ruby
class Article < ActiveRecord::Base
	after_save    :do_something
	after_destroy :do_something_else
	...
end

it "triggers do_something on save" do
	expect(@article).to receive(:do_something)
	@article.save
end

it "triggers do_something_else on destroy" do
	expect(@article).to receive(:do_something_else)
	@article.destroy
end

it "#do_something should work as expected" do
	# Actual tests for do_something method
end
```

Para testear su existencia, usa [Shoulda-callback-matchers](https://github.com/jdliss/shoulda-callback-matchers):
```ruby
describe Article do
	it { is_expected.to callback(:do_something).after(:save) }
end
```

## Skip callback in Factory

[Medium](https://blog.oozou.com/never-skip-a-callback-in-your-tests-28c552d70634)
Nunca los omitas.

[Stack Overflow](https://stackoverflow.com/questions/8751175/skip-callbacks-on-factory-girl-and-rspec)
No hay una propuesta concreta. Esta es una forma de hacerlo.
```ruby
FactoryGirl.define do
	factory :user do
		first_name "Luiz"
		last_name "Branco"
		#...

		after(:build) { |user| user.class.skip_callback(:create, :after, :run_something) }

		factory :user_with_run_something do
			after(:create) { |user| user.send(:run_something) }
		end
	end
end
```

# Difference between mock and stub 📰 

[Stack Overflow](https://stackoverflow.com/questions/7340808/whats-the-difference-between-mock-stub-and-factory-girl)

La mejor explicación está en este [artículo de Jason Swett](https://www.codewithjason.com/rspec-mocks-stubs-plain-english/).

> Stub: is a fake object that’s used in place of a real object for the purpose of getting the program to behave the way we need it to in order to test it.
> 
> Mock: is a fake object that’s used in place of a real object for the purpose of listening to the methods called on the mock object.
> 
> As I understand it, and to paint with a very broad brush, **Test Stubs help with inputs** and **Mock Objects help with outputs**.

# About RSpec `subject`

[Semaphore](https://semaphoreci.com/community/tutorials/rspec-subject-helpers-hooks-and-exception-handling)

[Diferencia entre subject y let](https://github.com/rubocop/rspec-style-guide/issues/6)

1. `subject` will automatically be assigned as an instance of the class being tested.
2. It allows for the use of "one liner" tests

```ruby
context "when something" do
	subject(:car) { FactoryGirl.create(:car) }

	it { is_expected.to_not be_running }
end
```

[rspec docs](https://relishapp.com/rspec/rspec-core/v/2-0/docs/subject/explicit-subject)

# Diferencia entre Feature, System y Request specs

En [rspec-rails](https://github.com/rspec/rspec-rails#system-specs-feature-specs-request-specswhats-the-difference).

**System specs**

> **acceptance tests**, **browser tests**, or **end-to-end tests**, system specs test the application from the perspective of a human client. The test code walks through a user’s browser interactions and the expectations revolve around page content.

**Feature specs**

> spec type for end-to-end testing(…)

Diferencia importante:

> feature specs **require non-trivial configuration** to get some important features working, like JavaScript testing or making sure each test runs with a fresh DB state. With system specs, this configuration is provided out-of-the-box.

**Request specs**

> Request specs are for testing the application from the perspective of a machine client. They begin with an HTTP request and end with the HTTP response, so they’re faster than feature specs, but do not examine your app’s UI or JavaScript.


# Sobre comparar fechas en tests

Visto en este [trino](https://twitter.com/stevepolitodsgn/status/1452652368953286665/photo/1).

La situación la encuentro por un test que escribió Steve y que explota por milisegundos:
```bash
   Failure/Error: expect(csv.map { |row| row[:created_at_0] }).to match_array([intake_a.created_at.iso8601, intake_b.created_at.iso8601])
         
      expected collection contained:  ["2022-01-17T15:18:03Z", "2022-01-17T15:18:04Z"]
      actual collection contained:    ["2022-01-17T15:18:03Z", "2022-01-17T15:18:03Z"]
      the missing elements were:      ["2022-01-17T15:18:04Z"]
      the extra elements were:        ["2022-01-17T15:18:03Z"]
```

En el trino, sugiere usar `asssert_in_delta` pero en RSpec eso se logra con el [*matcher*](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/be-within-matcher) [](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/be-within-matcher)`[be_within](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/be-within-matcher)` el cual tiene un uso un poco más complicado:

```ruby
expect(area_of_circle).to be_within(0.1).of(28.3)

expect(actual).to be_within(delta).of(expected)
```

pero para las fechas no me queda claro si puedo indicar un número para indicar milisegundos y tampoco lo probé.

Más sobre los [matchers de RSpec](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers).

