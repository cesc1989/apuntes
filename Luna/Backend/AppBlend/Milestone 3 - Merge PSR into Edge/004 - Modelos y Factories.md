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

Visto en este [commit](https://github.com/thoughtbot/factory_bot/commit/ef5c4ba49a182d051e39fe8ea3ef33591fff53ce).

# Tablas de ID bigint no tienen secuencia por Logical Replication

Para poder configurar el Logical Replication entre Forms DB y Edge tocaba dejar sin valor por defecto a las llaves primarias de las tablas. Eso significó quitar la secuencia por lo que ahora en las pruebas si se intentan crear registros para FormType explota por las restricciones de la bd:
```ruby
Failure/Error: ases = FactoryBot.create(:form_type, name: "ASES", acronym: "ases")

ActiveRecord::NotNullViolation:
  PG::NotNullViolation: ERROR:  null value in column "id" of relation "form_types" violates not-null constraint
  DETAIL:  Failing row contains (null, ASES, ases, 2024-10-28 20:51:21.855568, 2024-10-28 20:51:21.855568).
# /Users/francisco/.gem/ruby/3.1.0/gems/bullet-7.0.1/lib/bullet/active_record70.rb:6:in `_create_record'
```

Para poder avanzar me tocó poner una secuencia en el factory para el `id`:
```ruby
FactoryBot.define do
  factory :form_type, class: "PatientSelfReport::FormType" do
    sequence(:id) { |n| n }
  end
end
```

Pero esto a su vez genera problemas al intentar correr múltiples pruebas si no se reinicia la base de datos de pruebas.

> [!Note]
> En Grimoire pasa igual. La tabla clinic_payer [no tiene secuencia](https://github.com/lunacare/grimoire/blob/omega/priv/repo/migrations/20240120105000_unpk_clinic_payers.exs#L90) en el ID.

La solución es una mezcla de definir el `id` en una secuencia y usar el callback `initialize_with`.

Ejemplo para `OptionChoice`:
```ruby
# frozen_string_literal: true

FactoryBot.define do
  factory :option_choice, class: "PatientSelfReport::OptionChoice" do
    question

    sequence(:id) { |n| n }
    sequence(:label) { |n| "Mild #{n}" }
    value { 2 }

    initialize_with { PatientSelfReport::OptionChoice.find_or_create_by(id: id) }
  end
end
```

Visto en este sitio -> https://ploegert.gitbook.io/til/programmy/rails/find-or-create-a-record-with-factory-bot

