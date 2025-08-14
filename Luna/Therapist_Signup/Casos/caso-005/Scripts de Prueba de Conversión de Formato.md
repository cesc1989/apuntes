# Scripts de Prueba de Conversión de Formato

## Conversión a milisegundos

Estos son los scripts que usé para probar las diferentes formas de convertir la fecha (DateTime) al formato en milisegundos que espera HubSpot.

Fechas para probar:
```ruby
date_string = "2025-08-11 22:30:00 -0700"
date_string = "2025-08-12 23:59:00 -0500"
date_string = "2025-08-12"

ap "Testing with date string: #{date_string}"
puts
puts
```

### Fecha sin conversión de zona en milisegundos e iso8601

Esta es la forma original de la implementación para la conversión a milisegundos. Además está la otra opción que es convertir a cadena en iso8601.
```ruby
ap "Date original"
date = Time.parse(date_string)
ap date

ap "Date original convertida a milisegundos epoch"
res1 = Time.utc(date.year, date.month, date.day).strftime("%s%3N")
ap res1
puts

ap "Date original en iso8601"
res11 = Time.utc(date.year, date.month, date.day).iso8601
ap res11
puts
puts
```


### Fecha en UTC en milisegundos e iso8601

Este es para obtener la fecha en UTC y ver los resultados.
```ruby
ap "Date en UTC"
utc_date = date.utc
ap utc_date

ap "Date en UTC convertida a milisegundos epoch"
res2 = Time.utc(utc_date.year, utc_date.month, utc_date.day).strftime("%s%3N")
ap res2

ap "Date UTC en iso8601"
res22 = Time.utc(utc_date.year, utc_date.month, utc_date.day).iso8601
ap res22
puts
puts
```

> [!Note]
> Esta forma no sirvió porque básicamente es lo que hace la base de datos. Tomar un datetime y guardarlo en UTC.


### Fecha en zona horaria

La intención aquí es pasar la fecha a la zona horaria del usuario pero eso no lo podemos hacer en Therapist Signup.

Este ejemplo tampoco sirve mucho porque `in_time_zone` define por defecto `Time.zone` como parámetro.
```ruby
ap "Date en modo edge"
res3 = date.in_time_zone(Time.zone).to_date.to_datetime.to_i * 1000
ap res3
puts
puts
```

### Prueba de conversión de milisegundos a Fecha

Y esto es la verificación final de que los milisegundos se transforman en una fecha valida Y es el valor esperado.
```ruby
ap "#" * 50
ap "Ahora las fechas convertidas de milisegundos a date"
puts

ap "Date original en milisegundos pasada a Fecha:"
ap Time.at(res1.to_i / 1000).utc.strftime("%Y-%m-%d")

ap "Date en UTC en milisegundos pasada a Fecha:"
ap Time.at(res2.to_i / 1000).utc.strftime("%Y-%m-%d")

ap "Date en modo edge en milisegundos pasada a Fecha:"
ap Time.at(res3.to_i / 1000).utc.strftime("%Y-%m-%d")
```

## Validación de que Date.current con o sin TimeZone **NO** será igual

Estando ya en uso el campo `signup_date` el cual es de tipo Date **YO CREí QUE** no había necesidad de configurar el TimeZone en `application.rb`. Le pregunté a Claude y dijo que sí era necesario así que le pedí un script para comprobar eso.

Este es el script.

> [!Important]
> Este script está mal porque se está ejecutando `Date.current` en la misma hora que probé el script. Si lo corro a las 8AM, entonces el resultado siempre será favorable a mi hipótesis.
>
> Lo que en realidad necesito es probar el script con el servidor en una hora por la noche para poder verificar que no haya conversión horaria.

```ruby
date_string = "2025-08-11 22:30:00 -0700"  # 10:30pm Pacific

t1 = Therapist.where(hubspot_id: nil).first
puts "=== Testing Timezone Impact on Date.current ==="
puts

# Save original timezone
original_timezone = Time.zone.name
puts "Original timezone: #{original_timezone}"

# Test WITHOUT timezone config (UTC)
puts "\n--- WITHOUT timezone config (UTC) ---"
Time.zone = 'UTC'
puts "Time.zone now: #{Time.zone.name}"
puts "Date.current: #{Date.current}"

# Update therapist simulating the scenario
t1.update_columns(
  first_name: "Juan",
  last_name: "Without TZ Config",
  created_at: Time.parse(date_string),
  signup_date: Date.current  # This will be UTC date
)

puts "created_at: #{t1.created_at}"
puts "signup_date (UTC): #{t1.signup_date}"

# Test what HubSpot would receive
service_utc = HubspotContactService.new(therapist: t1, new_signup: true)
hubspot_date_utc = service_utc.send(:signup_form_date)
converted_utc = Time.at(hubspot_date_utc.to_i / 1000).utc.strftime("%Y-%m-%d")
puts "HubSpot would show: #{converted_utc}"

# Test WITH timezone config (Pacific)
puts "\n--- WITH timezone config (Pacific) ---"
Time.zone = 'America/Los_Angeles'
puts "Time.zone now: #{Time.zone.name}"
puts "Date.current: #{Date.current}"

t1.update_columns(
  last_name: "With TZ Config",
  signup_date: Date.current  # This will be Pacific date
)

puts "created_at: #{t1.created_at}"
puts "signup_date (Pacific): #{t1.signup_date}"

# Test what HubSpot would receive
service_pacific = HubspotContactService.new(therapist: t1, new_signup: true)
hubspot_date_pacific = service_pacific.send(:signup_form_date)
converted_pacific = Time.at(hubspot_date_pacific.to_i / 1000).utc.strftime("%Y-%m-%d")
puts "HubSpot would show: #{converted_pacific}"

# Restore original timezone
Time.zone = original_timezone
puts "\nRestored timezone to: #{Time.zone.name}"

puts "\n=== Results Summary ==="
puts "User signed up: Aug 11, 10:30pm Pacific"
puts "Without TZ config: HubSpot shows #{converted_utc}"
puts "With TZ config: HubSpot shows #{converted_pacific}"
puts "Difference: #{converted_utc != converted_pacific ? 'YES - dates differ!' : 'No difference'}"
```

Lo ejecuté en `rails console` y estos fueron los resultados:
```bash
=== Testing Timezone Impact on Date.current ===

Original timezone: UTC

--- WITHOUT timezone config (UTC) ---
Time.zone now: UTC
Date.current: 2025-08-13
  Therapist Update (2.4ms)  UPDATE "therapists" SET "first_name" = $1, "last_name" = $2, "created_at" = $3, "signup_date" = $4 WHERE "therapists"."id" = $5  [["first_name", "Juan"], ["last_name", "Without TZ Config"], ["created_at", "2025-08-12 05:30:00"], ["signup_date", "2025-08-13"], ["id", "7324ba4a-7c1d-418a-9f3e-7c34e725dd76"]]
created_at: 2025-08-12 05:30:00 UTC
signup_date (UTC): 2025-08-13
[DATE_TRANSFORMATION] Converting date: 2025-08-13 to a Hubspot format for 7324ba4a-7c1d-418a-9f3e-7c34e725dd76
HubSpot would show: 2025-08-13

--- WITH timezone config (Pacific) ---
Time.zone now: America/Los_Angeles
Date.current: 2025-08-13
  Therapist Update (0.5ms)  UPDATE "therapists" SET "last_name" = $1, "signup_date" = $2 WHERE "therapists"."id" = $3  [["last_name", "With TZ Config"], ["signup_date", "2025-08-13"], ["id", "7324ba4a-7c1d-418a-9f3e-7c34e725dd76"]]
created_at: 2025-08-12 05:30:00 UTC
signup_date (Pacific): 2025-08-13
[DATE_TRANSFORMATION] Converting date: 2025-08-13 to a Hubspot format for 7324ba4a-7c1d-418a-9f3e-7c34e725dd76
HubSpot would show: 2025-08-13

Restored timezone to: UTC

=== Results Summary ===
User signed up: Aug 11, 10:30pm Pacific
Without TZ config: HubSpot shows 2025-08-13
With TZ config: HubSpot shows 2025-08-13
Difference: No difference
```

> [!error]
> Todo lo anterior está mal. La configuración de zona horaria SÍ es necesaria.

No hubo diferencia alguna porque en palabras de Claude:

 The test was run on Aug 13, so regardless of timezone:
  - UTC: Aug 13
  - Pacific: Aug 13 (same calendar day)
 
 Since we're using Date.current at the time of signup, the timezone configuration doesn't matter because:
  1. User signs up → `Date.current` gets the current date
  2. Date is stored as a `t.date` field (no time component)
  3. HubSpot gets the stored date → always the date when signup occurred

