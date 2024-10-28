# Modelos y Factories: El Namespace cambia las cosas

Al tener los archivos de factories, modelos y pruebas dentro del namespace `PatientSelfReport`, cambian varias cosas en la definición de las pruebas y los factories.

## AggravatingActivity

Ejemplo con el modelo `AggravatingActivity`.

Así queda definido el modelo:
```ruby

module PatientSelfReport
  class AggravatingActivity
  end
end
```

Así queda la prueba:
```ruby
describe PatientSelfReport::AggravatingActivity, type: :model do
end
```

Y para soportar todo eso, así tiene que quedar el factory:
```ruby
FactoryBot.define do
  factory :aggravating_activity, class: "PatientSelfReport::AggravatingActivity" do
end
```

Si no se define la factory así, FactoryBot no es capaz de hallar la clase:
```bash
NameError:
       uninitialized constant Form

             Object.const_get(camel_cased_word)
```