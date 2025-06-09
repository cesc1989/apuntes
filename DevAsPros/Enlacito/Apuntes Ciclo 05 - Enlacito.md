# Apuntes Ciclo 05 -  Enlacito

## Personalizar vista previa del enlace en RR.SS y Mensajería

Para hacer lo de mostrar una vista previa personalizada como hace TinyZap toca con truco al procesar la petición.

![[001.tinyzap.enlace.personalizado.png]]

La clave está en leer el User-Agent de la petición.

Si se logra reconocer un bot de alguna red social o canal de mensajería, se retorna una página HTML que tiene las etiquetas Open Graph con los valores personalizados.

Si no se reconoce ningún bot, se hace una redirección 302.

### Obtener título automáticamente

Por un lado (con ayuda de DeepSeek) hice un scrapper para visitar la URL, extraer el título y guardarlo en el campo `title` de la tabla `links`.

### Cargar página para Bots

Para mostrar la página a los bots, chatgpt me sugirió esto:
```ruby
BOT_USER_AGENTS = /discordbot|facebookexternalhit|twitterbot|linkedinbot|slackbot|TelegramBot|WhatsApp/i

if bot_request?
	render "redirects/bot_preview", layout: false
else
	redirect_to @link.long_url, allow_other_host: true
end

private

def bot_request?
	request.user_agent.to_s =~ BOT_USER_AGENTS
end
```

Para comprobar esto, tanto en producción como en local, puedo usar curl.

Para probar que la petición la hace un bot:
```bash
curl -i -A "Discordbot/2.0" http://localhost:3005/cashflow

HTTP/1.1 200 OK

<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Cash Flow</title>

  <meta property="og:title" content="Cash Flow" />

  <meta property="og:url" content="http://localhost:3005/cashflow" />

  <meta http-equiv="refresh" content="0;url=https://cashflow.devaspros.com/" />
</head>
<body>
  <script>
    window.location.href = "https://cashflow.devaspros.com/";
  </script>
</body>
</html>
```

Devuelve código 200 y el HTML que está en la vista con las etiquetas personalizadas.

Para probar una petición normal, es decir, una persona abre el enlace:
```bash
curl -i http://localhost:3005/cashflow
HTTP/1.1 302 Found
```

Hace redirección con respuesta 302.

#### Para probar el bot de Twitter

```bash
curl -i -A "Twitterbot" https://enlacito.co/zKTIXJ
```