# ResubmitMp: resubmit to MSO desde consola

Helper para pegar en `heroku run rails c` (producción). Recibe el **omid** o el **name** del member period, valida, hace preview y, con `execute: true`, ejecuta `Salesforce::ResubmitToMso` con salidas a color.

## Uso

```ruby
# solo preview (por defecto)
ResubmitMp.run("01a0ace8-3620-7893-96d7-036874833395")

# preview + confirmación 'y' + resubmit
ResubmitMp.run("MP-012345", execute: true)

ResubmitMp.run(omid, execute: true, overrides: {prescription_strength: 10, weeks_of_supply: 4})

# aceptar el mismo med_id
ResubmitMp.run(omid, execute: true, allow_same_med: true)
```

## Validaciones

| Paso | ✖ Error (aborta, sin cambios) | ⚠ Advertencia |
|---|---|---|
| MP | No existe por omid ni por name, o hay varios con el mismo name | No es el MP más reciente de la cuenta |
| Estado | `PharmacyOrderShipped/Delivered`, `Canceled`, `MissedCheckin`, `ImportedFromOntraport` | `WaitingOnPharmacyConfirmation` / `PharmacyOrderConfirmed`: la orden en la pharmacy hay que cerrarla aparte |
| CEs | Más de un `Scheduled`, o ningún CE tiene recomendación con productos | Hay `Pending` (el resubmit los cancela). Si hay un `Scheduled`, se usa ese |
| Preview | Hay `validation_errors`, o no hay medicamento recomendado | — |
| Mismo `med_id` | Aborta salvo que pases `allow_same_med: true` | Con `allow_same_med: true` solo avisa |

Al ejecutar: muestra el cambio de estado del MP, la recomendación nueva, el medicamento y el CE nuevo, y lista los pendientes (cerrar la orden anterior en la pharmacy y el caso del CE cancelado en CareValidate/Beluga). Todo corre en la transacción del servicio, así que un error no deja nada a medias.

Para cambiar dosis o duración usa `overrides` (`prescription_strength`, `weeks_of_supply`), no `selected_med_id`: MedPicker debe elegir el medicamento.

## Código

```ruby
module ResubmitMp
  class Abort < StandardError; end

  # Ya en pharmacy: el resubmit no cancela la orden existente, hay que hacerlo a mano
  PHARMACY_STATUSES = %w[WaitingOnPharmacyConfirmation PharmacyOrderConfirmed].freeze
  # El medicamento ya salió o el MP está cerrado: no resubmitir
  BLOCKED_STATUSES = %w[PharmacyOrderShipped PharmacyOrderDelivered Canceled MissedCheckin ImportedFromOntraport].freeze

  module_function

  def run(identifier, execute: false, overrides: {}, allow_same_med: false)
    mp = find_member_period!(identifier.to_s.strip)
    check_member_period!(mp)
    ce = pick_clinical_encounter!(mp)
    old_rec = ce.med_picker_recommendation
    preview = run_preview!(ce, old_rec, overrides, allow_same_med)

    unless execute
      info "Fue solo el preview. Para ejecutar: ResubmitMp.run(#{mp.omid.inspect}, execute: true#{", overrides: #{overrides.inspect}" if overrides.present?})"
      return preview
    end

    confirm!(mp, ce)
    result = Salesforce::ResubmitToMso.execute(clinical_encounter: ce, overrides:)
    report_result(mp, old_rec, result)
    result
  rescue Abort => e
    error e.message
    nil
  rescue Patient::Checkin::ClinicalEncounterScheduler::NotSchedulable => e
    error "No se pudo agendar el CE (#{e.class.name.demodulize}): #{e.message}. Se revirtió todo."
    nil
  rescue Salesforce::ResubmitToMso::NoExistingRecommendationError,
    Salesforce::ResubmitToMso::NoMatchingProduct,
    Salesforce::ResubmitToMso::MedicationNotFoundError => e
    error "#{e.class.name.demodulize}: #{e.message}"
    nil
  rescue => e
    error "Error inesperado #{e.class}: #{e.message}"
    puts e.backtrace.first(5).map { "    #{it}" }
    nil
  end

  # --- pasos -------------------------------------------------------------

  def find_member_period!(identifier)
    raise Abort, "Falta el omid o el name del member period" if identifier.blank?

    header "Member period"
    mp = Salesforce::MemberPeriod.find_by(omid: identifier)
    unless mp
      matches = Salesforce::MemberPeriod.where(name: identifier).to_a
      raise Abort, "No existe un member period con omid o name #{identifier.inspect}" if matches.empty?
      raise Abort, "Hay #{matches.size} member periods con name #{identifier.inspect}: #{matches.map(&:omid).join(", ")}. Usa el omid." if matches.size > 1
      mp = matches.first
    end

    account = mp.account
    info "MP:      #{mp.name} (#{mp.omid})"
    info "Cuenta:  #{account&.name} <#{account&.person_email}>"
    info "Estado:  #{mp.status}"
    mp
  end

  def check_member_period!(mp)
    raise Abort, "El MP no tiene cuenta asociada" unless mp.account
    raise Abort, "El MP está en #{mp.status}; no se resubmite en ese estado" if BLOCKED_STATUSES.include?(mp.status)

    latest = mp.account.latest_member_period
    if latest && latest.omid != mp.omid
      warning "Este no es el MP más reciente de la cuenta (el último es #{latest.name} / #{latest.omid}, #{latest.status})"
    end

    if PHARMACY_STATUSES.include?(mp.status)
      warning "El MP está en #{mp.status}: ya hay una orden en la pharmacy. El resubmit NO la cancela; ciérrala con la pharmacy aparte."
    end
  end

  def pick_clinical_encounter!(mp)
    header "Clinical encounters"
    ces = mp.clinical_encounters
      .includes(med_picker_recommendation: {selected_products: [:product, :recommended_medication]})
      .order(Arel.sql("createddate ASC NULLS LAST, id ASC"))
      .to_a
    raise Abort, "El MP no tiene clinical encounters" if ces.empty?

    ces.each do |ce|
      puts "    #{ce.omid}  #{ce.status.to_s.ljust(10)} #{ce.source_system.to_s.ljust(14)} #{ce.created_at&.to_date}  rec=#{ce.med_picker_recommendation&.omid || "-"}"
    end

    scheduled = ces.select { it.status == "Scheduled" }
    raise Abort, "Hay #{scheduled.size} CEs en Scheduled (#{scheduled.map(&:omid).join(", ")}); hay que revisarlo a mano" if scheduled.size > 1

    pending = ces.select { it.status == "Pending" }
    warning "Hay #{pending.size} CE(s) en Pending; el resubmit los cancela" if pending.any?

    ce =
      if scheduled.one?
        warning "Se usa el CE Scheduled #{scheduled.first.omid} (si no, el agendamiento falla con 'A scheduled clinical encounter already exists')"
        scheduled.first
      else
        ces.reverse.find(&:med_picker_recommendation)
      end

    raise Abort, "Ningún CE tiene recomendación de MedPicker" unless ce&.med_picker_recommendation
    raise Abort, "La recomendación #{ce.med_picker_recommendation.omid} no tiene productos seleccionados" if ce.med_picker_recommendation.selected_products.empty?

    ok "CE objetivo: #{ce.omid} (#{ce.status}), recomendación #{ce.med_picker_recommendation.omid}"
    ce
  end

  def run_preview!(ce, old_rec, overrides, allow_same_med)
    header "Preview"
    info "Overrides: #{overrides.inspect}" if overrides.present?

    preview = Salesforce::ResubmitToMso.preview(clinical_encounter: ce, overrides:)
    if preview.validation_errors.any?
      preview.validation_errors.each { error it }
      raise Abort, "El preview falló; no se ejecuta nada"
    end

    old_meds = old_rec.selected_products.filter_map(&:recommended_medication)
    new_med = preview.recommended_medication

    puts "    Anterior:  #{old_meds.map { describe(it) }.join("\n               ").presence || "(sin medicamento)"}"
    puts "    Nuevo:     #{describe(new_med)}"
    puts "    Producto:  #{preview.matching_product&.name}"
    puts "    Opciones:  #{preview.possible_medications.size} medicamentos posibles"

    raise Abort, "MedPicker no devolvió medicamento recomendado" unless new_med

    if old_meds.map(&:med_id).include?(new_med.med_id)
      message = "MedPicker eligió el MISMO med_id (#{new_med.med_id}). Si estaba agotado, la orden se va a atascar de nuevo."
      raise Abort, "#{message} Usa allow_same_med: true si es intencional." unless allow_same_med
      warning message
    else
      ok "Medicamento distinto al anterior"
    end

    preview
  end

  def confirm!(mp, ce)
    header "Confirmación"
    warning "Se cancela el CE #{ce.omid}, se crea una recomendación nueva, el MP pasa a ReadyToCreateVisit y se agenda un CE nuevo."
    print "\e[1m¿Resubmitir #{mp.name}? Escribe 'y' para continuar: \e[0m"
    raise Abort, "Cancelado por el usuario; no se hizo ningún cambio" unless $stdin.gets.to_s.strip == "y"
  end

  def report_result(mp, old_rec, result)
    header "Resultado"
    mp.reload
    new_rec = result[:new_recommendation]
    new_meds = new_rec.selected_products.filter_map(&:recommended_medication)
    new_ce = mp.clinical_encounters.where(status: %w[Pending Scheduled]).order(Arel.sql("createddate DESC NULLS FIRST, id DESC")).first

    status_line = "Estado MP: #{result[:previous_status]} → #{mp.status}"
    (mp.status == "ReadyToCreateVisit") ? ok(status_line) : warning(status_line)
    ok "Recomendación nueva: #{new_rec.omid} (#{new_rec.status})"
    new_meds.each { ok "Medicamento: #{describe(it)}" }

    if result[:clinical_encounter_scheduled] && new_ce
      ok "CE nuevo: #{new_ce.omid} (#{new_ce.status}, #{new_ce.source_system})"
    else
      error "No se encontró CE nuevo en Pending/Scheduled; revisar"
    end

    if (old_rec.selected_products.filter_map { it.recommended_medication&.med_id } & new_meds.map(&:med_id)).any?
      warning "El med_id quedó igual al anterior"
    end

    if PHARMACY_STATUSES.include?(result[:previous_status])
      warning "Pendiente: cancelar o cerrar la orden anterior con la pharmacy (el MP estaba en #{result[:previous_status]})"
    end
    warning "Pendiente: cerrar en el proveedor (CareValidate/Beluga) el caso del CE cancelado si sigue abierto"
  end

  # --- helpers -----------------------------------------------------------

  def describe(med)
    return "(ninguno)" unless med
    "#{med.med_id} | #{med.description || med.name} | #{med.rx_strength}mg | #{med.product&.name}"
  end

  def paint(code, msg) = puts("\e[#{code}m#{msg}\e[0m")
  def header(msg) = paint("1;35", "\n== #{msg} ==")
  def info(msg) = paint(36, "• #{msg}")
  def ok(msg) = paint(32, "✔ #{msg}")
  def warning(msg) = paint(33, "⚠ #{msg}")
  def error(msg) = paint(31, "✖ #{msg}")
end
```
