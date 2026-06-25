# Código para solucionar casos en Orderlymeds

## Crear Nuevo Member Period

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

## Actualizar campo `workos_user_nk` para solucionar "Oops error"

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