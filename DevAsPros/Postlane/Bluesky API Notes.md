# Notas y Enlaces de la API de Bluesky

## com.atproto.server.createSession

Docs: https://docs.bsky.app/docs/api/com-atproto-server-create-session

Petición:
```bash
curl --request POST \
  --url https://bsky.social/xrpc/com.atproto.server.createSession \
  --header 'content-type: application/json' \
  --data '{
  "identifier": "elcoshinita.bsky.social",
  "password": "APP-PASSWORD"
}'
```

> [!Important]
> Hay que enviar el handle completo.

## Sobre Rate Limits

Docs: https://docs.bsky.app/docs/advanced-guides/rate-limits

### Resumen

Las que interesan para Postlane.

Overall API Requests (all endpoints)

- Rate limited by IP
- 3000 per 5 minutes

`com.atproto.server.createSession`

- Measured per account
- 30 per 5 minutes
- 300 per day