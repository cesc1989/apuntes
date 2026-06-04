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

La respuesta incluye:
```json
{
  "did": "did:plc:uqghwogmcebjpfzj5agrxa7i",
  "handle": "elcoshinita.bsky.social",
  "accessJwt": "eyJ0eXAiOiJhdCtqd3QiLCJhbGciOiJFUzI1NksifQ.eyJzY29wZSI6ImNvbS5hdHByb3RvLmFwcFBhc3MiLCJzdWIiOiJkaWQ6cGxjOnVxZ2h3b2dtY2VianBmemo1YWdyeGE3aSIsImlhdCI6MTc4MDQxNzAyNSwiZXhwIjoxNzgwNDI0MjI1LCJhdWQiOiJkaWQ6d2ViOnN0cm9waGFyaWEudXMtd2VzdC5ob3N0LmJza3kubmV0d29yayJ9.ZpNSCA0GpNIdjoGvatdSwhREsesJiefjJ-E54EKe1XN9YEj8Y5IxjeatpSIOWXHq_hu6XdUCPRWEnB5ymqW7XQ",
  "refreshJwt": "eyJ0eXAiOiJyZWZyZXNoK2p3dCIsImFsZyI6IkVTMjU2SyJ9.eyJzY29wZSI6ImNvbS5hdHByb3RvLnJlZnJlc2giLCJzdWIiOiJkaWQ6cGxjOnVxZ2h3b2dtY2VianBmemo1YWdyeGE3aSIsImF1ZCI6ImRpZDp3ZWI6YnNreS5zb2NpYWwiLCJqdGkiOiJWYWFyNy9PRmxxcVRjdWFKRzVwSUhCUFVyanBlTUtuY0Z4YnJ6dXI1SmFRIiwiaWF0IjoxNzgwNDE3MDI1LCJleHAiOjE3ODgxOTMwMjV9.2tikYloSZHYoTlWJ7A7DnQ8IecYBWyrB_LjFXOkL7NSrw8OT4paP5vE473Mwz6IbhI7rHbj6GFGwdBtiHsh_Xg",
}
```

Como valores necesarios para otras peticiones como `repo.createRecord` o `server.refreshSession`.

## com.atproto.server.refreshSession

Docs: https://docs.bsky.app/docs/api/com-atproto-server-refresh-session

Este es para obtener un nuevo `accessJwt` para hacer petición de `repo.createRecord`. No necesita body solo hay que mandar el Auth con el `refreshJwt`.

```bash
curl --request POST \
  --url https://bsky.social/xrpc/com.atproto.server.refreshSession \
  --header 'authorization: Bearer refreshJwt'
```

Responde igual que `repo.createSession`.

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