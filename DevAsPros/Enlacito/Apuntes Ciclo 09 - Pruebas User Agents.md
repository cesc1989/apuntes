# Apuntes Ciclo 09 - Pruebas User Agents

Comandos locales para probar peticiones con los diferentes User-Agents para que cuente o no el click.

## UA para que no cuenten los clicks

### DiscordBot

```bash
curl -i -A "Discordbot/2.0" http://localhost:3005/cashflow
```

### TwitterBot

```bash
curl -i -A "Twitterbot" http://localhost:3005/QjHd80
```

## User-Agents para contar clicks

### Firefox

```bash
curl -i -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0" http://localhost:3005/QjHd80
```

### Safari (macos/ios)

```bash
curl -i -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Safari/605.1.15" http://localhost:3005/QjHd80
```

### Brave

```bash
curl -i -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36" http://localhost:3005/QjHd80
```