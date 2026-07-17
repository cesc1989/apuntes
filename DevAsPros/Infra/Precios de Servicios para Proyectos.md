# Precios de Servicios de Infraestructura para Proyectos

Servicios necesarios para la operación de cualquier proyecto.

Son:

- Captura de errores: Sentry o Bugsink
- Resend
- Hospedaje de archivos: AWS S3 o Cloudflare
- Hospedaje tipo Git Push: Netlify o Vercel o Heroku
- VPS: Linode o Host Hatch
- Monitoreo: Cronitor o Uptima Kuma

## Captura de Errores 🐞

### Sentry

> [!Important]
> Creo que Sentry es muy caro para equipos y proyectos pequeños. Mejor una instancia de Bugsink que es mucho más económica con características similares.

La capa gratuita es más que suficiente para un proyecto en sus años iniciales. Solo puede acceder un usuario a la vez.

**Para usuarios ilimitados son 29 USD/mes.**

### Bugsink

Web: https://www.bugsink.com/

Alrededor de 3.5 USD/mes en Pikapods. Sin límites de usuarios ni proyectos.

## Envío de Correos 📧

## Resend

> [!Important]
> Por el precio, podría ser mejor hacer varias cuentas por cada servicio o reusar algún subdominio ya verificado.
>
> Esto porque quedaría subutilizada la suscripción para un servicio apenas empezando y para el alcance que visionamos.

> [!Note]
> Para proyectos pequeños con tráfico estos límites son difícil de alcanzar.

La capa gratuita ofrece 3K correos al mes. Solo permite verificar un dominio.

**Para más dominios y más correos son 20 USD/mes.**

## Hospedaje de Archivos 🪣

Para hospedar imágenes, vídeos y/o archivos.

### Cloudflare R2

> [!Note]
> Para proyectos pequeños con tráfico estos límites son difícil de alcanzar.

La capa gratuita da 10 GB/mes.

### AWS S3

> [!Note]
> Para proyectos pequeños con tráfico estos límites son difícil de alcanzar.

Los primeros 50 TB/mes son a 0.023 USD por GB. O sea, 1 USD.

## Hospedaje tipo Git Push 🌏

### Netlify

Caro.

### Vercel

Caro.

### Heroku

> [!Note]
> No se puede usar SQLite.

Lo más básico que sería web dyno + postgres serían 10 USD/mes.

> [!Important]
> El precio de 10 USD/mes sería usando Solid Trifecta para aprovechar la base de datos y no necesitar Redis. Esto en la configuración donde la misma base de datos se usa para cola, caché y "socket".

## VPS: Linode o Host Hatch 🖥️

### Linode

El VPS más barato cuesta 5 USD/mes. Incluye:

- 1 GB de RAM
- 1 CPU
- 25 GB de espacio
- 1 TB de transferencia

### Host Hatch

El VPS más barato cuesta 4 USD/mes. Incluye:

- 2 GB RAM DDR4
- 1 CPU
- 10 GB espacio NVMe
- 1 TB de transferencia

## Monitoreo

### Cronitor

Web: https://cronitor.io/pricing

La capa gratuita da hasta 5 monitores. Más que suficiente.

**Si se quiere tener más, cuesta 2 USD/mes.** Además, **cada usuario costaría 5 USD/mes.**

### Uptime Kuma Self Hosted

En Pikapods.

**Costaría 4 USD/mes.** Sin ningún límite.