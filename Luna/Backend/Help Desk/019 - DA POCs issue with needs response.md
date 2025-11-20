# 019 - DA POCs incorrectly marked as needs_response no

Etiquetas: #luna_help_desk 

Relacionado: [[008 - Update POC Fax and Violation Logic 🟢]]

Caso EDG-3032

Reporte:
> Direct Access POCs are being faxed, _but marked as needs response no, even though the signature page is sending with the fax_.

## Resumen

En el caso relacionado uno de los problemas fue que para Direct Access se estaban creando POCs con `needs_response = false` para Virginia. Pasa que en VA el `resolution_mode` es `referral` así que el fix que hice para ese caso fue:

```ruby
def plan_of_care_needs_response?
	state = appointment.region.state
	rule_type = appointment.episode.referral_type.to_sym
	resolution_mode = state.plan_of_care_rules_config.dig(rule_type.to_s, "resolution_mode")

	return false if resolution_mode == "referral"

	# Default behavior: needs response unless discharged
	!appointment.discharged?
end
```

Lo cual tenía solo un poco de sentido porque antes el valor para `needs_response` siempre se calculaba en base a si el appointment fue discharged:

```ruby
needs_response: !chart.appointment.discharged?
```

Ahora para este nuevo reporte la solución según Claudio debería ser algo al estilo:
```ruby
return false if resolution_mode == "referral" && appointment.initial_visit?
```

Sin embargo, eso está mal porque en el caso anterior el problema se dio con Initial Visits de Virginia.

## Preguntas

Procedí a preguntarle a Indy lo siguiente:

**Q: What do you mean with the "signature page" that is sent in the fax? Can you show one example?**
A: It’s the last page in the fax that the MD signs. Typically if something is marked as needs response no, this page is not present.


**Q: For DA POCs, what does the "referral" resolution mode means?**
A: Direct access can be resolved by a signed POC or referral. If a referral gets attached to the “Referral file” field in the care plan, it will resolve the POC and move the pt to “Referred” status instead of DA.

### Signature Page

Pude ver que los PDFs generados llevaban una página al final que tiene espacios en blancos para que el physician firme y devuelva vía fax.

> [!Important]
> Mi pregunta es: ¿Todos los templates tienen esa página?
>
> Respuesta: Sí. Todos lo que es POC usa el mismo template el cual contiene la página de signature.

Encuentro que en `app/services/plans_of_care/plan_of_care_fax_pdf_service.rb` está esto:
```ruby
# Alpha google template ids
GOOGLE_DOC_TEMPLATE_MAP = {
  medicare_initial: "GOOGLE_ID",
  medicare_standard: "GOOGLE_ID",
  medicare_progress: "GOOGLE_ID",
  medicare_discharge: "GOOGLE_ID",
  direct_access_initial: "GOOGLE_ID",
  direct_access_standard: "GOOGLE_ID",
  direct_access_progress: "GOOGLE_ID",
  direct_access_discharge: "GOOGLE_ID",
  fax_on_demand_initial: "GOOGLE_ID",
  fax_on_demand_standard: "GOOGLE_ID",
  fax_on_demand_progress: "GOOGLE_ID",
  fax_on_demand_discharge: "GOOGLE_ID",
  referred_initial: "GOOGLE_ID",
  referred_standard: "GOOGLE_ID",
  referred_progress: "GOOGLE_ID",
  referred_discharge: "GOOGLE_ID",
  payer_initial: "GOOGLE_ID",
  payer_standard: "GOOGLE_ID",
  payer_progress: "GOOGLE_ID",
  payer_discharge: "GOOGLE_ID",
  workers_comp_initial: "GOOGLE_ID",
  workers_comp_standard: "GOOGLE_ID",
  workers_comp_progress: "GOOGLE_ID",
  workers_comp_discharge: "GOOGLE_ID",
  temp_storage_folder_id: "GOOGLE_ID"
}.freeze
```

Pedí acceso para comprobar si hay alguna plantilla sin la página de signature que dice Indy.