# Scripts de Prueba de Conversión de Formato

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

## Fecha sin conversión de zona en milisegundos e iso8601

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


## Fecha en UTC en milisegundos e iso8601

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


## Fecha en zona horaria

La intención aquí es pasar la fecha a la zona horaria del usuario pero eso no lo podemos hacer en Therapist Signup.

Este ejemplo tampoco sirve mucho porque `in_time_zone` define por defecto `Time.zone` como parámetro.
```ruby
ap "Date en modo edge"
res3 = date.in_time_zone(Time.zone).to_date.to_datetime.to_i * 1000
ap res3
puts
puts
```

## Prueba de conversión de milisegundos a Fecha

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