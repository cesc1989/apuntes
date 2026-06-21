# Precios de Servicios de Infraestructura para Proyectos

Servicios necesarios para la operación de cualquier proyecto.

Son:

- Sentry
- Resend
- AWS S3 o Cloudflare
- Netlify o Vercel
- VPS: Linode o Host Hatch

## Sentry 🐞

La capa gratuita es más que suficiente para un proyecto en sus años iniciales. Solo puede acceder un usuario a la vez.

**Para usuarios ilimitados son 29 USD/mes.**

## Resend 📧

> [!Info]
> Para proyectos pequeños con tráfico estos límites son difícil de alcanzar.

La capa gratuita ofrece 3K correos al mes. Solo permite verificar un dominio.

**Para más dominios y más correos son 20 USD/mes.**

## AWS S3 o Cloudflare 🪣

Para hospedar imágenes, vídeos y/o archivos.

### Cloudflare R2

> [!Info]
> Para proyectos pequeños con tráfico estos límites son difícil de alcanzar.

La capa gratuita da 10 GB/mes.

### AWS S3

> [!Info]
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
> El precio de 10 USD/mes sería usando Solid Trifecta para aprovechar la base de datos y no necesitar Redis.

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