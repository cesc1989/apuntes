# Apuntes Ciclo 25

# Correos desde Mailgun llegan a spam

Los correos diarios empezaron a llegar solo a spam. Había pensado cambiar al modo API de Mailgun pero según DeepSeek y [ChatGPT](https://chatgpt.com/share/688537ca-4994-800d-ab09-27706cdbf8a5) eso no servirá porque la IP del plan gratuito tendrá mala reputación.

## IP Dedicada en Mailgun

Estaba revisando y encontré una información al respecto:

> Dedicated IPs help isolate your sending reputation and improve your deliverability when sending a large volume of messages. We recommend sending at least 50,000 messages a week to maintain a strong reputation (...).
>
> Mailgun recommends one dedicated IP address for every 1 million messages you are sending each month.

## Planes de Mailgun

Más bajo es de 15 usd al mes. Es muy caro para solo enviar un correo para el asunto de las Tareas Recurrentes.

## Alternativas a Mailgun

Según ChatGPT Resend.

Probé esta y la documentación ayudó mucho con la configuración en Namecheap. Ya configuré todo e hice una prueba. Fue exitosa.