# KX Modifier - Agregar el Modificador

Ya llegué a la hora de la verdad. Voy a agregar el modificador a los CPT codes si se necesita.

Alexis sugirió dos formas en el modelo AutoChart:

1. Agregar un método `procedures_with_kx` donde se modifique el JSON que tiene los códigos
	1. Luego usar este método donde haga falta
2. Modificar `self.cpt_code` para agregarle el modificador

Veamos cuantas veces aparece bien sea el campo JSON o el método de clase.

## `procedures_with_kx` - Campo JSON

El campo JSONB se llama: `procedures_form`. Así se ve uno cualquiera:
```ruby
AutoChart.first.procedures_form
{"gait_training"=>15,
 "manual_therapy"=>10,
 "pt_re_evaluation"=>0,
 "therapeutic_activity"=>0,
 "therapeutic_exercise"=>10,
 "canalith_repositioning"=>0,
 "neuromuscular_re_education"=>0,
 "pt_evaluation_low_complexity"=>0,
 "pt_evaluation_high_complexity"=>0,
 "pt_evaluation_moderate_complexity"=>0,
 "patient_education_self_care_management"=>0,
 "physical_performance_test_or_measurement"=>5}
```

Todos los posibles procedures se cargan desde la base de datos:
```ruby
YAMLHelper.load_common_config("procedures_performed_form")

{
  "pt_evaluation_low_complexity" => {
    "title" => "PT Evaluation, Low Complexity",
    "html_title" => "PT Evaluation, <span class=\"text-green-500\">Low</span> Complexity"
  }
}
```

La sugerencia del método de instancia es esto:
```ruby
class AutoChart
	def procedures_with_kx
    return procedures_form unless chart&.requires_kx_modifier?

    procedures_form.transform_values do |procedure|
      procedure.merge("title" => "#{procedure['title']}-KX")
    end
  end
end
```

## `self.cpt_code` - Método de clase

La otra alternativa que sugiere Alexis es modificar el método de clase `cpt_code` para que devuelva el modificador KX si lo necesita.

```ruby
class AutoChart
  def self.cpt_code(procedure_key)
    billing_procedure_codes[procedure_key]["cpt_code"] + "KX"
  end
end
```

Creo que este se alinea mucho mejor con lo que hay que hacer porque se está más cerca del CPT Code.

> [!note]
> El CPT Code es lo que debe llevar el modificador KX:
> En el PRD dice: "*All additional CTP Codes within a patient’s care plan should have a KX Modifier Code included*"

La carga de los procedures con CPT Codes se da desde la base de datos:
```ruby
YAMLHelper.load_common_config("billing_procedure_codes")
{
  "procedure_keys" => {
    "pt_evaluation_low_complexity" => {
      "title" => "PT Evaluation, Low Complexity",
      "cpt_code" => "97161"
    }
  }
}
```


Al buscar `AutoChart.cpt_code` en Sublime aparecen resultados en 6 archivos:

- app/admin/clinical/charts.rb
- app/candid/candid/mappers/appointment_to_candid_encounter.rb
- app/models/concerns/billing_unit_calculations.rb
- app/workers/therapist_signed_chart_pdf_generator_worker.rb
- app/workers/therapist_unsigned_chart_pdf_generator_worker.rb
- app/workers/athena/candid_claims_writer_worker.rb

Si voy por la segunda alternativa, esos son los archivos donde debo empezar a mirar si tiene sentido aplicar esta solución.