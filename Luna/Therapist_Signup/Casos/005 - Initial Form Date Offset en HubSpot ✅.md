# 005 - Propiedad Initial Form Date muestra valor desfasado en la UI de HubSpot

Documentos relacionados:

- [[Integración_con_HubSpot_-_Lecciones#Sobre el formato de los campos `Date` y `DateTime` para HubSpot]]
- [[Hubspot_y_las_fechas_date_is_more_than_1,000_year]]

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
> ==In HubSpot, date properties always display the specific date they are set to, regardless of the time zone setting of the portal or the user==.

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

También pude comprobar que tanto en Alpha y Omega la cuenta de HubSpot está configurada en la zona horaria Pacific UTC -07.

## ¿Será el problema guardar las fechas en UTC?

> [!Important]
> Sí. Este es el problema.
>
> Como no se definía el time zone a nivel de Rails, la fecha se guarda en UTC Y se presenta en UTC.
>
> Guardar en UTC está bien pero se necesita hacer la conversión para evitar que se adelante un día al enviar a HubSpot.

Therapist Signup nunca definió una time zone por defecto. Para los Contactos afectados, así está el valor de `created_at`.

Skye Harry:
```
Fecha mala: 03/31/2025
Fecha esperada: 3/30/2025

created_at: 2025-03-31 00:37:36 UTC
```

Celine Emery:
```
Fecha mala: 01/30/2025
Fecha esperada: 1/29/2025

created_at: 2025-01-30 02:58:04 UTC
```

Melanie Wokwicz:
```
Fecha mala: 01/30/2025
Fecha esperada: 1/29/2025

created_at: 2025-01-30 04:47:16 UTC
```

Cindy Okpegbue:
```
Fecha mala: 08/05/2025
Fecha esperada: 8/4/2025

created_at: 2022-04-19 03:28:40 UTC
```

Queda claro que como el `created_at` está en UTC, ya la fecha viajó al día siguiente, entonces lo que se manda a HubSpot es la fecha en el día siguiente para los casos de PDT.

# Primera Solución 🔴

> [!Note]
> Esto sirve bien para usuarios en Pacific pero podría causar el mismo problema para usuarios en otras zonas horarias.

Según Claude, la primera solución está en:

- Configurar el TimeZone en Rails

```ruby
def timezone_aware_formated_date(date, therapist_id: nil)
	local_date = date.in_time_zone(Time.zone)
	Time.utc(local_date.year, local_date.month, local_date.day).strftime("%s%3N")
end
```

- Usar `in_time_zone` para la conversión de la fecha cuando viaja a HubSpot

```ruby
# config/application.rb

config.time_zone = "America/Los_Angeles"
```

Esto me parece bien porque el campo `created_at` es un timestamp (datetime) y causa todos estos problemas al tener valores para la hora.

En cambio, otros campos que usan la función original son solo tipo fecha. Ver [[Campos Date y DateTime que se envian a HubSpot]]

# Segunda Solución 🟢

Crear un nuevo campo que sea de tipo `Date`. Con ese campo se evita todo ese lío de la zona horaria porque simplemente se guarda la fecha sin más.

Documentos relacionados:

- Scripts de pruebas de conversión a milisegundos -> [[Scripts de Prueba de Conversión de Formato#Conversión a milisegundos]]
- Script para comprobar que no es necesario configurar la zona horaria -> [[Scripts de Prueba de Conversión de Formato#Validación de que Date.current con o sin TimeZone será igual]]