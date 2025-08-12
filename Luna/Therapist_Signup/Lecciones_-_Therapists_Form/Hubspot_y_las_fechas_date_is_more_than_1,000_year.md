# Hubspot y las fechas: date is more than 1,000 years ago or 1,000 years from
Sobre este error que ha pasado en varias ocasiones:
```
Hubspot::RequestError
Response body: {"validationResults":[{"isValid":false,"message":"-62044444800000 is more than 1,000 years ago or 1,000 years from now.","error":"INVALID_DATE","name":"malpractice_liability_insurance_expiration"}],"status":"error","message":"Property values were not valid","correlationId":"6a28f4f2-b54d-47ef-b32b-d29a40e44ffd"}
```

Este es el código que tengo para convertir las fechas según lo que necesita Hubspot recibir:
```ruby
def formated_date(date)
		return nil if date.nil?

		Time.utc(date.year, date.month, date.day).strftime('%s%3N')
end
```

En este documento intentaré entender qué pasa en esta función, cómo funciona `strftime` y por qué pasa el error.

## Enlaces

- [Miliseconds in Ruby](https://dev.to/bdavidxyz/milliseconds-in-ruby-3g9l)
- [Rails DateTime docs](https://api.rubyonrails.org/classes/DateTime.html)

## ¿Qué hace la función?

La función:
```ruby
def formated_date(date)
		return nil if date.nil?

		Time.utc(date.year, date.month, date.day).strftime('%s%3N')
end
```

Lo que hace es devolver una fecha en milisegundos desde la fecha EPOCH, en UTC y con la parte de hora, minutos y segundos apuntando a las 12 de la medianoche.

Si queremos enviar la fecha 1ero de Mayo de 2015, tendría que enviarla así:
```
# Fecha: 01/05/2015
# En UTC, medianoche: 01-05-2015 00:00:00 UTC
# Como necesita Hubspot: 1430438400000
```


## ¿Qué hace `strftime`?

Para conseguir esos milisegundos, necesitamos obtenerlos de la fecha y eso lo logramos usando `strftime`:
```ruby
d = Date.parse('2015-05-01')
date = Time.utc(d.year, d.month, d.day) # La pasamos a UTC

sin_mili = date.strftime('%s')
# => 1430438400

actual = date.strftime('%s%3N')
# => 1430438400000

v2 = date.strftime('%3N')
# => 000
```

Según la documentación de `strftime` veamos que hace cada opción:
```
   -> %s
    Seconds since the Epoch:
      %s - Number of seconds since 1970-01-01 00:00:00 UTC.
    
    -> %3N
    %N - Fractional seconds digits, default is 9 digits (nanosecond)
              %3N  millisecond (3 digits)
              %6N  microsecond (6 digits)
              %9N  nanosecond (9 digits)
              %12N picosecond (12 digits)
              %15N femtosecond (15 digits)
              %18N attosecond (18 digits)
              %21N zeptosecond (21 digits)
              %24N yoctosecond (24 digits)
           The digits under the specified length are truncated to avoid
           carry up.
```

Ya vemos que `%s` trae la fecha en segundos desde [Epoch](https://en.wikipedia.org/wiki/Epoch_(computing)). Y `3%N` nos da los milisegundos de la fecha.

Enlaces:

- [strftime docs](https://devdocs.io/ruby~3/time#method-i-strftime)
- [Ruby strftime, short and long story](https://www.bootrails.com/blog/ruby-strftime-short-and-long-story/)

Entonces si volvemos a la conversión de 1er de Mayo de 2015:

```ruby
sin_mili = date.strftime('%s')
# => 1430438400
```

- Solo son los segundos desde la fecha Epoch.
- No sirve para lo que necesita Hubspot.

```ruby
actual = date.strftime('%s%3N')
# => 1430438400000
```
  
- Nos da los segundos desde la fecha Epoc en milisegundos.
- Lo que sí necesita Hubspot.

```ruby
v2 = date.strftime('%3N')
# => 000
```

- Solo devuelve unos milisegundos.
- No sirve para nada.


## ¿Por qué se da este error?

Al parecer, al calcular la fecha la cantidad de milisegundos es muy arbitraria y está causando que la fecha sea mucho más de 1000 años en el futuro o en el pasado.

Cuando intenté convertir la fecha dada en milisegundos en el error, obtuve esto:
```ruby
# Fecha del error en milisegundos: -62044444800000
dme = -62044444800000
dme_to_date = Time.at(dme / 1000).utc
# => 0003-11-22 00:00:00 UTC
```

Para convertir tiempo en milisegundos a fecha, hay que primero dividirlo entre mil (para pasarlo a segundos) y luego sí pasarlo a la función `Time.at`.

Enlaces:

- [Time.at](https://ruby-doc.org/core-2.7.1/Time.html#method-c-at)
- [How to convert long/milliseconds to DateTime?](https://www.ruby-forum.com/t/re-how-to-convert-long-milliseconds-to-datetime/73488)

