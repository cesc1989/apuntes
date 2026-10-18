# Apuntes Apex Ciclo 011

## Sentry IpSpoofAttackError

### Qué pasó

- **Error:** `ActionDispatch::RemoteIp::IpSpoofAttackError: IP spoofing attack?!`
- **Último evento:** 2026-10-10 20:40 UTC
- **Causa:** un bot/escáner (IP real `45.138.12.47`) manda cabeceras `Client-IP` y `X-Forwarded-For` con IPs inventadas que cambian en cada petición.
- nginx reenviaba esas cabeceras a Puma:
  - `Client-IP` pasaba sin tocar.
  - `X-Forwarded-For` usaba `$proxy_add_x_forwarded_for`, que conserva lo que manda el cliente y añade la IP real al final.
- Rails ve las dos cabeceras sin coincidir, no sabe cuál es la IP del cliente y lanza la excepción en `request.remote_ip` (al escribir "Started GET…" en el log). El cliente recibe un 500.

### Impacto

- **Seguridad:** ninguno. Rails rechazó la petición y `remote_ip` no fue engañado.
- **Ruido:** un evento en Sentry por cada petición del bot.
- **Riesgo para usuarios reales:** algunos proxies corporativos o de operadores móviles añaden `Client-IP`; esos usuarios también recibían un 500.

### Fix

Commit `4f2f2cb` en `config/nginx.enlacito.production.conf`:

```diff
     proxy_set_header X-Real-IP $remote_addr;
-    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
+    proxy_set_header X-Forwarded-For $remote_addr;
+    proxy_set_header Client-IP "";
     proxy_set_header X-Forwarded-Proto https;
```

- `X-Forwarded-For $remote_addr`: Rails recibe solo la IP real del cliente.
- `Client-IP ""`: con valor vacío, nginx no envía la cabecera a Puma.
- Aplicado a mano en el servidor; `sudo nginx -t` OK.
- El deploy enlaza el archivo (`scripts/003_after_deploy.sh`) y recarga nginx (`scripts/004_application_start.sh`).

### Cómo probar

```bash
curl -i -H "Client-IP: 1.2.3.4" -H "X-Forwarded-For: 5.6.7.8" https://enlacito.co/
```

- **Esperado:** respuesta normal (200/302), no un 500.
- En Sentry no debe aparecer un evento nuevo de ENLACITO-7

### Alternativas descartadas

- `config.action_dispatch.ip_spoofing_check = false`: quita una protección en vez de arreglar la causa.
- Excluir la excepción en Sentry: solo oculta el ruido; los usuarios con `Client-IP` seguirían recibiendo un 500.