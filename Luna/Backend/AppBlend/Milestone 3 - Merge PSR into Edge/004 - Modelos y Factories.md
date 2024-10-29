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

Documentación sobre este callback -> https://thoughtbot.github.io/factory_bot/ref/build-and-create.html#initialize_with

## Generar ID de llave primaria manualmente

Continuando el caso anterior, resulta que al crear el `patient_form_detail` hay que ponerle su ID de forma manual así:
```ruby
manual_id = SecureRandom.random_number(1 << 63)
create_patient_form_detail(id: manual_id, internal_id: internal_id)
```

> [!warning]
> **Pregunta**: ¿El método de asociación no agrega el ID?

# Configurando Recursos Nested por la limitante de Logical Replication

> [!important]
> Este problema también se da por la falta de la secuencia en la llave primaria debido a que se quitó para poder configurar la Logical Replication.

Ha habido dos casos y creo que serán cada vez más frecuentes.

> [!note]
> Se pudo mejorar le manejo de esto, de manera global, usando un Concern que genere el ID de los modelos. Más abajo la implementación.

## En PatientForm

Así está originalmente este código para configurar un form nuevo a crear junto con un nuevo paciente:
```ruby
  def setup_form_attrs
    return if @forms_attributes.blank?

    attrs = @forms_attributes&.first&.merge!(progress_type: Form::ONBOARDING_PROGRESS)
    @patient.forms.clear if @patient.new_record?
    @patient.forms_attributes = [attrs]
  end
```

Es sencillo. Se asigna un array de atributos de uno o más `Form` al método `forms_attributes`.

Sin embargo, con el problema de que no hay secuencia y toca controlar la creación de los IDs, tocó cambiarlo así:
```ruby
    def setup_form_attrs
      return if @forms_attributes.blank?

      @patient.forms.clear if @patient.new_record?

      # This way diverts from the original approach because otherwise would think the assignment is to update the form.
      # What we need is to build new records.
      @forms_attributes.each do |form_attrs|
        manual_id = SecureRandom.random_number(1 << 63)
        form_attrs.merge!(id: manual_id, patient_id: @patient_id, progress_type: PatientSelfReport::Form::ONBOARDING_PROGRESS)

        @patient.forms.build(form_attrs)
      end
    end
```

Y toca así porque mediante `@patient.forms.build` es la forma en que logramos decirle a Rails que el registro es nuevo. De lo contrario (usando `forms_attributes`) si hay algún ID, Rails creerá que se trata de una operación `update`.

En el caso de la operación Update tiraría el error de que no puede encontrar un registro para otro registro.

Este es un ejemplo para otro caso similar:
```bash
Failure/Error: @intake_form.medications_attributes = prepare_medications_attributes

     ActiveRecord::RecordNotFound:
       Couldn't find PatientSelfReport::Medication with ID=1 for PatientSelfReport::IntakeForm with ID=5085303943573520300
```

## En IntakeFormBuilder

En esta clase, también hay creación anidada de recursos. Para el caso de medications la cosa luce así originalmente:
```ruby
def build
	@intake_form.attributes = prepare_intake_attributes
	@intake_form.medications_attributes = prepare_medications_attributes
	@intake_form.surgeries_attributes = prepare_surgeries_attributes
	@intake_form
end

private

def prepare_medications_attributes
	return [] unless @medications

	meds = @medications.map do |med|
		next if med[:name].nil?

		med
	end

	meds.compact
end
```

Pero ahora en Edge tocó cambiarlo así:
```ruby
def build
	@intake_form_id = SecureRandom.random_number(1 << 63)
	@intake_form.attributes = prepare_intake_attributes.merge(id: @intake_form_id)
	prepare_medications_attributes
	@intake_form.surgeries_attributes = prepare_surgeries_attributes

	@intake_form
end

private

def prepare_medications_attributes
	return [] unless @medications

	meds = @medications.map do |med|
		next if med[:name].nil?

		manual_id = SecureRandom.random_number(1 << 63)
		med.merge(id: manual_id)
	end

	@intake_form.medications.build(meds.compact)
end
```

Por los mismos motivos que en el caso de `PatientForm`. Rails creerá que se trata de un update si no se asigna mediante el método de asociación `@intake_form.medications.build`.

# Concern para generar el primary key de los modelos 🎉

Por recomendación de Alexis, configuré un concern que se encarga de asignar el ID a los modelos de Patient Self Report:
```ruby
# frozen_string_literal: true

# PatientSelfReport tables were all LRed without the default value for the
# primery key. Generate one while LR is enabled or else we'd get not-null constraint errors.
module PatientSelfReportPrimaryKey
  extend ActiveSupport::Concern

  included do
    before_create :generate_unique_bigint_id
  end

  private

  def generate_unique_bigint_id
    return if self.id.present?

    begin
      self.id = SecureRandom.random_number(1 << 63)
    end while self.class.exists?(id: self.id)
  end
end
```

De esta forma me ahorro tener que andar haciendo tantos cambios y generar el ID para cada registro en cada parte.