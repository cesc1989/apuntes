# Notas y Enlaces de la API de Bluesky

## com.atproto.server.createSession

Docs: https://docs.bsky.app/docs/api/com-atproto-server-create-session

Petición:
```bash
curl --request POST \
  --url https://bsky.social/xrpc/com.atproto.server.createSession \
  --header 'content-type: application/json' \
  --data '{
  "identifier": "elcoshinita",
  "password": "APP-PASSWORD"
}'
```