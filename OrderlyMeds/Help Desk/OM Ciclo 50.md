# OM Ciclo 50

> [!Info]
> Del Jueves 2 de Julio al Miércoles 15 de Julio.

## Caso OM-9839 - New MP diferente dosis 🟡ℹ️

Etiquetas: #om_new_mp_different_dosage

Dice:
> This patient mistakenly filled out the current dosage (first check in) and ended up being prescribed with 2.5mg when been on 15mg.
>
> reset link so can check in asap for 15mg.

Había creado un nuevo MP pero, dicen, quedó con la misma dosis de 2.5mg.

> [!Note]
> Hay instrucciones en el Notion sobre algo relacionado con el MedPicker Recommendations asociadas en Ontraport pero no me queda claro. Pregunté a Wilkes.
>
> Me dijo que:
> > You don't have to do that. You should reset the checkin so that the customer reorder through the normal checkin process.



## Caso OM-9790 - Script error con `needs_requested_medpicker_data` 🟡ℹ️

Etiquetas: #om_needs_requested_medpicker_data

Caso de Stuck in Submitted que al hacer el resubmit el Script queda en Outcome "Script Error" y el CareValidate::Request en estado `needs_requested_medpicker_data`.

En Ontraport, en la sección "OMFS Data" del Script dice:
> No matching recommendations for these patient preferences.

Intenté correr:
```ruby
result = CareValidate::FixResubmitOntraportWebhook.call(
  request_id: "019f23ba-6d29-7dac-818a-af2c6b672855"
)
```

pero devolvió el error que aparece en Success:
```ruby
{error:
  "No MedPicker recommendation found for CareValidate for patientPreference: [{\"name\" => \"OM Ultra: Tirzepatide - $399/month - 4 Injections\", \"numberMonths\" => \"1\", \"strength\" => \"None\", \"change\" => \"Weight Loss is Going Great\", \"ingredients\" => \"B3,B5\", \"titrateup\" => \"I want to increase dosage every month\", \"medIdOverride\" => \"\", \"refills\" => \"0\"}]"}
```

Luego intenté correr:
```ruby
CareValidate::GetMedpickerDataJob.new.perform(
  webhook.id,
  request.id,
  request.script_nk
)
```

Pero también dio error:
```
No matching recommendations for these patient preferences.
```

> [!Note]
> Para poder correr `CareValidate::GetMedpickerDataJob` el campo `source_event` del webhook debe ser `care_validate_checkin` para que al llamar a `CareValidate::ProcessRequestJob` se ejecute la parte en que se encola a `CareValidate::SendCheckinJob`. Job que también es llamado en el proceso `FixResubmitOntraportWebhook`. 