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

# Encontrar Chart y AutoChart para ver el Modificador en Luxe

Para poder probar en Luxe necesito encontrar un AutoChart apto. Además, el Appointment de ese AutoChart debe tener todos los modelos necesarios para esto: `MedicareDollarThresholdStatus` y `MedicareCarePlanMedicalNecessityResponse`.

## Buscar AutoChart

Primero, busco un AutoChart cuyo campo `procedure_form` no esté vacío:
```sql
select ac.id,
  ac.appointment_id as "appt id",
  epi.id as "episode",
  ac.procedures_form
from auto_charts ac
join appointments appo on appo.id = ac.appointment_id
join episodes epi on epi.id = appo.episode_id
where ac.procedures_form is not null;
```

Cogemos el `episode_id` de cualquiera para empezar a crear los registros y a actualizar el appointment.

## Mover Episode a Medicare insurance

Necesitamos que el Episode sea Medicare:
```ruby
epi = Episode.find("101d7cf6-9468-4a46-8a52-4c67de15c0d2")
epi.update(insurance_id: 3)
```

## Actualizar el Appointment `initial_visit`

En alpha, normalmente los datos que encuentro son viejos. Puede ser que el appointment esté cancelado. Para cambiar eso usamos esta query:
```ruby
appo_1 = epi.appointments.first

appo_1.update(state: :pending, scheduled_date: appo_1.scheduled_date + 1.day)
```

## Crear MDTS y marcar como excedido

Como son registros viejos, hay muchas reglas que tal vez no se cumplan. En ese caso, al actualizar el appointment puede que no se cree el registro MDTS. Para corregir eso lo podemos crear manualmente:
```ruby
MedicareDollarThresholdStatus.create(
  patient: epi.patient,
  effective_from: DateTime.new(2024, 1, 1).beginning_of_year,
  effective_until: DateTime.new(2024, 12, 31).end_of_year
)
```

Y luego actualizamos el `threshold_exceeded`:
```ruby
epi.medicare_dollar_threshold_status.update(threshold_exceeded:true)
```

## Crear Medical Necessity aprobada

Finalmente, creamos el registro Medical Necessity con estado aprobado para que todo termine de conectar:
```ruby
MedicareCarePlanMedicalNecessityResponse.create(
  medicare_dollar_threshold_status: epi.medicare_dollar_threshold_status,
  care_plan: epi,
  therapist: epi.initial_visit.therapist,
  medical_necessity_state: :approved
)
```

## Ubicamos el ID del Chart

Con esta otra query, usando el ID del episode, debe salir el ID del chart con el cual ir a Luxe.
```sql
select epi.id as "episode",
  insu."key" as "insurance",
  mdts.threshold_exceeded as "exceeded",
  mcpmnr.medical_necessity_state as "necessity",
  appo.id as "appt id",
  c.id as "chart_id",
  ac.id as "auto chart id"
from episodes epi
join insurances insu on insu.id = epi.insurance_id
join medicare_care_plan_medical_necessity_responses mcpmnr on mcpmnr.care_plan_id = epi.id
join medicare_dollar_threshold_statuses mdts on mdts.id = mcpmnr.medicare_dollar_threshold_status_id
join appointments appo on appo.episode_id = epi.id
left join charts c on c.appointment_id = appo.id
left join auto_charts ac on ac.appointment_id = appo.id
where insu."key" = 'medicare'
and epi.id = '101d7cf6-9468-4a46-8a52-4c67de15c0d2';
```

Ya con ese ID podemos ir a la página del Chart en Luxe y ver los cambios en acción.