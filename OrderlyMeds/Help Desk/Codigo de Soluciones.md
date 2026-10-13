# Código para solucionar casos en Orderlymeds

## Crear Nuevo Member Period

Etiquetas: #om_new_mp 

```ruby
def new_member_period(email, checkin_due_date: Date.today + 14.days)
  account = Account.where(email:).sole.salesforce_account
  previous_member_period = account.latest_member_period
  checkin_deadline_date = checkin_due_date + Salesforce::MemberPeriod::CHECKIN_GRACE_PERIOD
  new_lifecycle_stage = (previous_member_period.customer_type == "Employee") ? "NotApplicable" : "Existing"

  Salesforce::MemberPeriod.create!(
    account: previous_member_period.account,
    status: "ReadyForCheckin",
    customer_type: previous_member_period.customer_type,
    customer_lifecycle_stage: new_lifecycle_stage,
    loyalty_points: previous_member_period.loyalty_points,
    checkin_due_date: checkin_due_date,
    checkin_deadline_date: checkin_deadline_date
  )
end
```

Después se ejecuta la función pasando el correo:
```ruby
new_member_period("")
```

Mensaje post:
```
👋🏾 CX is ready for check-in.
```

## Cambiar CareValidate::Request a `needs_resubmission`

Etiquetas: #om_stuck_in_submitted 

```ruby
def to_resubmit_ontraport_script(request_id:)
  puts "Buscando CareValidate::Request con ID: #{request_id}"
  request = CareValidate::Request.find(request_id)

  puts "Request encontrado con state #{request.state}"

  puts "\n"
  puts "Actualizando Request a state needs_resubmission"
  request.update!(state: "needs_resubmission")
end
```

Ejecuta:
```ruby
to_resubmit_ontraport_script(request_id: "")
```

## Actualizar campo `workos_user_nk` para solucionar "Oops error" o errro HTTP ERROR 400

Etiquetas: #om_oops_error  #om_http_error_400

```ruby
def fix_workos_user_nk(email:, workos_user_id:)
  puts "Buscando account con email: #{email}"
  account = Account.find_by(email:)
  raise "Account no encontrado para email: #{email}" unless account

  puts "Account encontrado: id=#{account.id}, email=#{account.email}"
  puts "Valor actual de workos_user_nk: #{account.workos_user_nk.inspect}"

  puts "Actualizando workos_user_nk a: #{workos_user_id}"
  account.update!(workos_user_nk: workos_user_id)
  puts "workos_user_nk actualizado correctamente: #{account.reload.workos_user_nk}"

  puts "Creando Salesforce::CustomerUser para account id=#{account.id}"
  puts "  salesforce_account: #{account.salesforce_account&.id}"
  Salesforce::CustomerUser.create(
    salesforce_person_account: account.salesforce_account,
    local_account: account
  )
  puts "Salesforce::CustomerUser creado exitosamente"
end
```

Ejecuta pasando el correo y el id copiado desde WorkOS:
```ruby
fix_workos_user_nk(email: "", workos_user_id: "")
```

## Resincronizar Account a Salesforce para resolver error de NO_ACCESS o el Authentication Error

Etiquetas: #om_no_access_error  #om_authentication_error

```ruby
def fix_salesforce_customer_user(account_id:)
  puts "Buscando account con id: #{account_id}"
  account = Account.find(account_id)
  puts "Account encontrado: id=#{account.id}, email=#{account.email}"

  puts "salesforce_account: #{account.salesforce_account&.id}"
  puts "Creando Salesforce::CustomerUser para account id=#{account.id}"
  Salesforce::CustomerUser.create(
    salesforce_person_account: account.salesforce_account,
    local_account: account
  )
  puts "Salesforce::CustomerUser creado exitosamente"
end
```

Correr con:
```ruby
fix_salesforce_customer_user(account_id: "ACCOUNT_ID")
```

## Crear cuenta en WorkOS - Error de one-time code de WorkOS o Cannot Impersonate

Etiquetas: #om_works_code_error #om_cannot_impersonate

Esta depende si el Account está ligado a Ontraport o Salesforce.

Función:
```ruby
def create_workos_user(account_id:, source:)
  raise "source debe ser 'salesforce' o 'ontraport'" unless %w[salesforce ontraport].include?(source)

  puts "Buscando account con id: #{account_id}"
  account = Account.find(account_id)
  puts "Account encontrado: id=#{account.id}, email=#{account.email}"

  puts "Obteniendo datos desde: #{source}"
  email, first_name, last_name = case source
  when "salesforce"
    sf = account.salesforce_account
    puts "  salesforce_account id: #{sf&.id}"
    [sf.person_email, sf.first_name, sf.last_name]
  when "ontraport"
    contact = account.ontraport_contact
    puts "  contact id: #{contact&.id}"
    [account.email, contact.first_name, contact.last_name]
  end

  puts "Datos a usar — email: #{email}, first_name: #{first_name}, last_name: #{last_name}"

  puts "Creando WorkOS user..."
  workos_user = Workos::CreateCustomerUser.call(email:, first_name:, last_name:)
  puts "WorkOS user creado: id=#{workos_user.id}"

  puts "Actualizando workos_user_nk en account id=#{account.id}"
  account.update!(workos_user_nk: workos_user.id)
  puts "workos_user_nk actualizado correctamente: #{account.reload.workos_user_nk}"
end
```

Salesforce:
```ruby
create_workos_user(account_id: "ACCOUNT_ID", source: "salesforce")
```

Ontraport:
```ruby
create_workos_user(account_id: "ACCOUNT_ID", source: "ontraport")
```

## Para revisar el estado de un MemberPeriod que no llega a Pharmacy

Relacionado con #om_member_period_not_at_pharmacy 

```ruby
bundle = RxWrittenBundle.find("019f6d9e-facb-70b1-a1af-f2597653bfb5")
order = Order.find_by(case_id: bundle.case_id)

SmartPharma::ProcessOrder.call(order:, logger: Rails.logger)
```

## Cambiar estado de Medication Request a Completed

Relacionado con #om_missing_previous_medication 

Para pasarlo de Unknown o Cancelled a Completed.

```ruby
def complete_mr!(omid:)
  mr = Salesforce::MedicationRequest.find_by(omid__c: omid)

  if mr.nil?
    puts "❌ No se encontró MedicationRequest con omid__c=#{omid.inspect}"

    return
  end

  puts "Estado actual: #{mr.status.inspect}"

  if mr.status == Salesforce::MedicationRequest::STATUS_COMPLETED
    puts "ℹ️ Ya está en #{Salesforce::MedicationRequest::STATUS_COMPLETED}, no se hace nada"

    return
  end

  mr.update!(status: Salesforce::MedicationRequest::STATUS_COMPLETED)
  puts "✅ Actualizado"

  mr.reload
  puts "Estado final: #{mr.status}"
end
```

Llamado con:
```ruby
complete_mr!(omid: "OMID")
```

## Activar Starter Pack en Member Period

```ruby
def restart_member_period(name)
  puts "\n=== Restart Member Period ==="
  puts "Buscando MemberPeriod: #{name}"

  mp = Salesforce::MemberPeriod.find_by(name: name)

  unless mp
    puts "❌ No se encontró el MemberPeriod: #{name}"
    return
  end

  puts "✓ MemberPeriod encontrado: #{mp.name}"

  mp.update!(customer_lifecycle_stage: "Restarting")
  puts "✓ customer_lifecycle_stage → Restarting"

  check_in = mp.patient_checkins.last

  unless check_in
    puts "⚠️ No hay patient_checkin asociado. No se realizó ninguna actualización."
    return
  end

  puts "✓ Check-in encontrado: #{check_in.id}"

  check_in.update!(is_starter_plan_only: true)
  puts "✓ is_starter_plan_only → true"

  puts "=== Proceso completado ===\n"
end
```

Uso:
```ruby
restart_member_period("MP-00700678")
```


### Mensaje importante para poder activar el Starter Pack

> [!Important]
> > Please have cx only complete Check In and **stop once they arrive to the Check Out page**. When they're in Check Out page is when Starter Pack can be activated. Let me know so I can proceed. Thanks.


## Aplica "Delivered" a Member Period de Missing Orders

```ruby
def update_imported_outcome(member_period_omid, correct_outcome)
  puts "\n=== Update Member Period Outcome ==="
  puts "Buscando MemberPeriod por OMID: #{member_period_omid}"

  mp = Salesforce::MemberPeriod.find_by(omid__c: member_period_omid)

  unless mp
    puts "❌ No se encontró el MemberPeriod con OMID: #{member_period_omid}"
    return
  end

  puts "✓ MemberPeriod encontrado: #{mp.name}"
  puts "  Outcome actual: #{mp.ontraport_imported_outcome.inspect}"
  puts "  Nuevo outcome: #{correct_outcome}"

  mp.update!(ontraport_imported_outcome: correct_outcome)

  puts "✓ ontraport_imported_outcome actualizado correctamente"
  puts "=== Proceso completado ===\n"
end
```

Ejecuta:
```ruby
update_imported_outcome(
  "019fc547-66af-7cb7-9371-1355c5a48abe",
  "Delivered"
)
```

## Migrar Contacto de Ontraport a Salesforce

```ruby
def migrate_op_to_sf(account_id)
  log = ->(label, value) { puts "  #{label.to_s.ljust(22)} #{value.inspect}" }
  abort_with = ->(reason) { puts "\n❌ ABORTADO: #{reason}"; return nil }

  a = Account.find(account_id)
  puts "\n== Cuenta #{a.id} (#{a.email})"

  if a.migrated_to_salesforce?
    log.(:salesforce_account_nk, a.salesforce_account_nk)
    return abort_with.("ya está migrada a Salesforce")
  end

  contact = a.ontraport_contact
  return abort_with.("no tiene contacto de Ontraport") if contact.blank? || contact.nk.blank?
  cid = contact.nk
  log.(:ontraport_contact_id, cid)

  prescriber = a.prescriber
  group_id, flag = if prescriber.care_validate?
    [Salesforce::CareValidateMigrationRolloutJob::CARE_VALIDATE_GROUP_ID, :enable_care_validate_salesforce_migrations]
  else
    [Salesforce::BelugaHealthMigrationRolloutJob::BELUGA_HEALTH_GROUP_ID, :enable_salesforce_migrations]
  end
  log.(:prescriber, prescriber)
  log.(:feature_flag, flag)
  log.(:flag_enabled, Features.enabled?(flag))
  return abort_with.("flag #{flag} está apagado") unless Features.enabled?(flag)

  # Informativo: el job lo usa como filtro, aquí no bloquea
  in_group = begin
    RateLimited::OntraportApiClient.new(limiter: Sidekiq::Limiter.unlimited)
      .get_contacts_by_group(group_id, ids: cid.to_s, perform_all: 1).any?
  rescue => e
    "error: #{e.class}: #{e.message}"
  end
  log.(:"in_group_#{group_id}", in_group)

  ci = OntraportMigration.fetch_contact(cid)
  puts "\n== Scripts [id, outcome, next_consult, case_id]"
  ci.member_period_mappings.each { s = it.ontraport_script; puts "  #{[s.id, s.outcome, s.next_consult, s.case_id].inspect}" }

  unless ci.valid?
    pp ci.validation_errors
    return abort_with.("contact_import inválido (ver validation_errors arriba)")
  end
  log.(:contact_import_valid, true)

  puts "\n== Migrando..."
  attempts = 0
  begin
    attempts += 1
    Ontraport::Meta.with_rate_limiting(limiter: Sidekiq::Limiter.unlimited) do
      Salesforce::OntraportAccountImporter.call(
        ontraport_contact_id: cid,
        contact_import: ci,
        local_account: a,
        logger: Logger.new($stdout)
      )
    end
  rescue RuntimeError => e
    raise unless e.message.include?("sfid is not available yet") && attempts < 4
    puts "  sfid aún no sincroniza (intento #{attempts}/3), reintentando en 30s..."
    sleep 30
    retry
  end

  a.reload
  return abort_with.("el importer terminó pero salesforce_account_nk sigue vacío") unless a.migrated_to_salesforce?

  sa = a.salesforce_account
  puts "\n✅ Migrada"
  log.(:omid, sa.omid)
  log.(:sfid, sa.sfid)
  log.(:lightning_url, sa.lightning_url)
  puts "\n== MemberPeriods [status, checkin_due_date, ontraport_script_id]"
  Salesforce::MemberPeriod.where(account: sa).pluck(:status, :checkin_due_date, :ontraport_script_id).each { puts "  #{it.inspect}" }
  sa
end
```

Si fue exitoso, la salida sería `:imported`.

Cómo funciona:
- **Se detiene sin migrar** en cuatro casos: si la cuenta ya está migrada, si no tiene contacto de OP, si el flag del prescriber está apagado, o si `contact_import` es inválido (en ese caso imprime los `validation_errors`).
- **El grupo (859 o 987) solo se muestra** en la salida y no bloquea nada.
- **Si el `sfid` tarda en llegar**, reintenta hasta 3 veces cada 30s. Se puede hacer porque el importer es idempotente.
- **Al terminar** imprime el omid, el sfid, la URL de Lightning y los MemberPeriods. Si algo falla, devuelve `nil`; si sale bien, devuelve la `Salesforce::PersonAccount`.