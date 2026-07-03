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

## Cambiar CareValidate::Request a needs_resubmission

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

## Actualizar campo `workos_user_nk` para solucionar "Oops error"

Etiquetas: #om_oops_error 

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

## Resincronizar Account a Salesforce para resolver error de NO_ACCESS

Etiquetas: #om_no_access_error 

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

## Crear cuenta en WorkOS - Error de one-time code

Etiquetas: #om_one_time_code_error 

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
    contact = account.contact
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

Ejecuta con:
```ruby
create_workos_user(account_id: "ACCOUNT_ID", source: "salesforce")
# create_workos_user(account_id: "ACCOUNT_ID", source: "ontraport")
```