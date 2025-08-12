# 005 - Propiedad Initial Form Date muestra valor desfasado en la UI de HubSpot

Reporte:

> This therapists ==initial form date was set to 08/05/2025==, but **they signed on 08/04/2025 at 5:27pm**.

¿Por qué es importante que esto no pase?

> (...) it disrupted a few of the workflows that alert the Sherpa when a therapist signs.

Este es el caso de Omega:

![[005.offset.initial.form.date.png]]

Así logré replicar uno en Alpha:

![[005.offset.initial.alpha.png]]

## Contexto

Esta propiedad "Initial Form Date" (`signup_form_date`) es de tipo Date en HubSpot. Para guardar valores en este hay que enviar el valor en milisegundos apuntando a la medianoche.

Esto que dice HubSpot:
> Date properties will only store the date, and must be set to midnight UTC for the date you want.
> 
> For example, May 1 2015 would be 1430438400000 (01 May 2015 00:00:00 UTC). If you try to set a value that is not midnight UTC, you will receive an error.
> 
> _In HubSpot, date properties always display the specific date they are set to, regardless of the time zone setting of the portal or the user_.

Para hacer eso, en los inicios del proyecto Therapist Signup se implementó esta función:
```ruby
def formated_date(date, therapist_id: nil)
	return nil if date.nil?

	Time.utc(date.year, date.month, date.day).strftime("%s%3N")
end
```

La cual convierte una fecha en milisegundos en la media noche de la fecha. Ejemplo.

```ruby
# Fecha: 2025-08-11 22:30:00 -0700
"1754870400000"
```

## Primera Pasada

Inicialmente, Claude dijo que esto pasa porque se estaba extrayendo la fecha sin tener en cuenta el timezone.

Propuso esta función:
```ruby
def formated_date_utc(date, therapist_id: nil)
	return nil if date.nil?

	# Convert to UTC first, then extract date components to avoid timezone offset issues
	utc_date = date.utc
	Time.utc(utc_date.year, utc_date.month, utc_date.day).strftime("%s%3N")
end
```

Cuando reviso en la consola de Rails, puedo ver algunas diferencias para esto
```ruby
date = Time.parse("2025-08-11 22:30:00 -0700")  # 10:30pm Pacific
ap date
res1 = Time.utc(date.year, date.month, date.day).strftime("%s%3N")
ap res1

utc_date = date.utc
ap utc_date
res2 = Time.utc(utc_date.year, utc_date.month, utc_date.day).strftime("%s%3N")
ap res2

ap Time.at(res1.to_i / 1000).utc.strftime("%Y-%m-%d")
ap Time.at(res2.to_i / 1000).utc.strftime("%Y-%m-%d")
```

se devuelve lo siguiente:
```ruby
2025-08-11 22:30:00 -0700 # date
"1754870400000"           # res1. En milisegundos sin timezone

2025-08-12 05:30:00 UTC   # date.utc
"1754956800000"           # res2. En milisegundos CON timezone

"2025-08-11"              # convertir res1 a date
"2025-08-12"              # convertir res2 a date
```

> [!Important]
> Todo esto me convenció y al probarlo se vio funcionar normalmente aunque no pude replicar con certeza el caso.
> Sin embargo, noté que la aplicación no define un TimeZone. Los datos se guardan en UTC.
> 
> Con eso en mente, ¿era el TimeZone la causa del fallo?

## Segunda Pasada

Cuando reviso el TimeZone en alpha y local obtengo lo mismo:
```ruby
Time.zone.name
=> "UTC"
```

Estoy en la zona neutra que pide HubSpot para la conversión de la fecha. ¿Estaba mal Claude?

Al parecer sí.

> Since Rails is configured to use UTC timezone:
> 
> - `@therapist.created_at` is stored and retrieved as UTC
> - The original `formated_date` method works correctly because it's extracting date components from UTC time
