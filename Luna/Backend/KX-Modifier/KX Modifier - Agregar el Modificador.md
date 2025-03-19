# KX Modifier - Agregar el Modificador

Ya llegué a la hora de la verdad. Voy a agregar el modificador a los CPT codes si se necesita.

## 🟡 `procedures_with_kx` - Campo JSON 🟡

El campo JSONB se llama: `procedures_form`. Así se ve uno cualquiera:
```ruby
AutoChart.first.procedures_form
{
  "gait_training"=>15,
  "pt_evaluation_low_complexity"=>0,
  "pt_evaluation_high_complexity"=>0
}
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

No estoy seguro de que por aquí sea el camino porque:

- En el campo `procedures_form` no hay un código CPT. Solo un título de procedimiento y a eso no se le agrega el modificador KX, según he visto.
- En el hash cargado en el YAML tampoco hay código CPT. Solo el título del procedimiento. Pasa similar al punto anterior.


## 🔴 `self.cpt_code` - Método de clase 🔴

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
> 
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

Si bien en esta opción se está más cerca del CPT Code, no es donde debe aplicarse porque es un método de clase y no se tiene acceso a la instancia con la cual se hace la verificación.

## En AppointmentToCandidEncounter#service_lines_field

Aquí hay que agregarlo en la construcción de los parámetros según indica la [Referencia de la API de Candid](https://docs.joincandidhealth.com/api-reference/service-lines/v-2/create#request.body.modifiers).

```
modifiers: list of enums. Optional
```

En la clase:
```ruby
modifiers = appointment.chart.requires_kx_modifier? ? ["KX"] : []

# (...)

{
	procedure_code: "A9270",
	quantity: 1,
	units: "UN",
	diagnosis_pointers: [0],
	modifiers: modifiers
}
```

Meredith aprueba los cambios ahí y además me dio una explicación sobre partes de la API de Candid.

> I just realized that Candid's Service Lines model is going to impact you, since that is their underlying model you are modifying.
> 
> When we sync an `Appointment` to Candid, that becomes an "Encounter" in their system. Candid models chart info by saying an "Encounter" has many "Diagnoses" and many "Service Lines".
> 
> Their API is odd in that their Create Encounter API endpoint, those fields are different from their Update Encounter API endpoint. If you modify the diagnoses or any field that ends up on a Service Line, instead of using their CRUD endpoints for Encounters, you have to use their CRUD endpoints for Diagnoses and Service Lines.
> 
> The function that handles upserting Service Lines: `Candid::Client.refresh_candid_diagnoses_and_service_lines!`.
> 
> [Candid API docs: Service Lines](https://docs.joincandidhealth.com/api-reference/service-lines/v-2/)
>
> [Candid API docs: Encounters](https://docs.joincandidhealth.com/api-reference/encounters/v-4/) — notice in Update there is no `service_lines` field supported.

