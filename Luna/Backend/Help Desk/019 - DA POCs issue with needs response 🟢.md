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

## Conclusión 🟡

Entendí mal el requerimiento inicial de Indy cuando pidió actualizar las reglas de Fax y Violation para Direct Access.

El caso es EDG-2746. En la petición ella dijo:
> New laws are in effect in 3 states and we need POC logic to match.

En los comentarios en Linear ella dijo:
> **VA**: "Fax when IV is signed (no signature required)" + "No violation Alerts"

O sea que para el estado de Virginia esa es la regla ahora. No debe haber necesidad de firma para las IV (Initial Visits).

Por alguna razón no entendí o pensé que esa regla se aplicaría sola. Ahora entiendo mejor cómo va esto de los POCs, su generación y la generación del PDF.

# Solución: Dos Partes

La solución se da en dos partes:

1. Corregir la función `Chart#plan_of_care_needs_response?` para que haga el chequeo con el estado y no con el modo de resolución.
2. Cambiar la generación del PDF para que cuando `needs_response = false` se elimine la última página, es decir, la que pide la firma del documento.

## 1) needs_response = false para Virginia IVs

Esto fue lo que dijo Indy en el primer issue:

- "TX, KS, and VA all had law updates for direct access."
- VA:
	- Fax when IV is signed (no signature required)
	- No violation Alerts

Cuando hice los cambios no entendí esa parte de la exclusividad de Virginia. Por eso el arreglo que hice para los reportes pasados afectaron los de este nuevo reporte.

### Primer Intento: solo código ❌

Esta solución:
```ruby
return false if state.postal_abbreviation == "VA" && appointment.initial_visit?
```

Anthony dijo que estaba mal porque todo sobre los POCs se configura por base de datos para evitar soluciones ad-hoc.

### Segundo Intento: config en DB

Así que ahora voy a probar guardando la configuración para calcular este valor en el campo `plan_of_care_rules_config` de la tabla `states`.

Esta es la configuración que voy a probar:
```json
{
  "direct_access":
  {
    "fax_rules":
    {
      "anchor": "initial_visit",
      "trigger":
      {
        "strategy": "chart_signature"
      }
    },
    "resolution_mode": "referral",
    "initial_visit_physician_response_required": false
  }
}
```

## 2) Quitar la signatura page para VA IVs

Como el POC se creará con `needs_response = false` entonces hay que quitar la última página del PDF que se envía al physician.

# Pruebas

## Solución `needs_response` para Virginia

Tengo dos scripts en mis documentos. El primero es para crear datos necesarios para probar la creación del POC. El segundo usa esos datos para crear un POC y revisa que el valor para `needs_response` sea false.

Se ejecutan así desde una consola de Rails siempre que no los mueva de lugar o cambie el nombre del archivo.

Primero crear datos para Virginia:
```ruby
load '/Users/francisco/Documents/Documents/luna-dev-files/edge/help-desk/edg-2746-pocs-rules/initial-visit-issue/01.create_test_poc_data.rb'
```

Luego correr el script de verificación
```ruby
load '/Users/francisco/Documents/Documents/luna-dev-files/edge/help-desk/edg-2746-pocs-rules/initial-visit-issue/02.reproduce_poc_needs_response_bug_simple.rb'
```

## Prueba de generación de POC para otros casos

Tengo este otro script para probar los casos por fuera de Virginia para otros resultados de POCs.

Se ejecuta con desde una consola de Rails
```ruby
load '/Users/francisco/Documents/Documents/luna-dev-files/edge/help-desk/edg-3032-pocs-2/01.test_non_initial_visit_poc.rb'
```

Para el cual puedo ver que se crea el POC con `needs_response = true`
```
INSERT INTO "plans_of_care" ("chart_id", "messenger", "needs_response", "created_at", "updated_at", "physician_id", "original_referral_type", "episode_id") VALUES ($1, $2, $3, $4, $5, $6, $7, $8) RETURNING "id"  [["chart_id", 123570], ["messenger", "phaxio"], ["needs_response", true]

POC Details:
  ID: 34cb0344-fc61-45d5-9195-707b8292457c
  needs_response: true
```

# Apartado Especial ℹ️

Creo que una vez más me confundí con lo que dijo Indy y por no actuar pronto no aclaré.

Indy dijo:
> Doesn’t need a response if it’s a discharge. That’s actually a different pdf that gets sent

O sea que para la pregunta que hizo Ryan, ya el initial visit discharged caería en este condición.

También dijo:
> If it’s the only appt, but the care plan is active, it still gets sent as YES

Esto no se cumple porque un initial visit que es discharged se daría para un care plan discharged o tratamiento completado.

Iba a poner este código pero creo que eso no está bien. Lo dejo acá por si acaso.

```ruby
# modelo
  def plan_of_care_needs_response?
    state = appointment.region.state

    return true if state.postal_abbreviation == "VA" &&
                   appointment.initial_visit? &&
                   appointment.discharged?

    return state.poc_iv_physician_response_required(rule_type: :direct_access) if appointment.initial_visit?

    !appointment.discharged?
  end

# pruebas
  context "with DA discharged IV in Virginia as the only appt" do
    it "requires response" do
      va_state = create(:state, postal_abbreviation: "VA")
      va_state.plan_of_care_rules_config["direct_access"] = {
        "initial_visit_physician_response_required" => false
      }
      va_state.save!

      region = create(:region, state: va_state)
      patient = create(:patient)
      episode = create(:episode, patient: patient, referral_url: nil)
      initial_visit = create(
        :appointment,
        episode: episode,
        region: region,
        visit_type: "initial",
        discharged: true
      )
      chart = create(:chart, appointment: initial_visit)

      expect(chart.plan_of_care_needs_response?).to be true
    end
  end
```
