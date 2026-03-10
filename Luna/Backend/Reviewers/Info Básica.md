# Información Básica sobre Reviewers

## Enlaces

Explicado en Notion [https://www.notion.so/getluna/Echo-07fe49e5a4c848a5be9d9669fa2b7573](https://www.notion.so/getluna/Echo-07fe49e5a4c848a5be9d9669fa2b7573)

Reviewers en Alpha: [https://echo.alpha.getluna.com/reviewers/sign_in](https://echo.alpha.getluna.com/reviewers/sign_in)

Reviewers en Local: [http://localhost:3000/reviewers/sign_in](http://localhost:3000/reviewers/sign_in)

## Configuración de Entorno

### Frontend

Necesito clonar la aplicación frontend que vive en https://github.com/lunacare/echo-frontend

Una vez clonado ejecutar el comando `yarn install`. Para correr el servidor se corre `yarn dev`.

#### ENVS

Hay cuatro en `.env.example`:
```bash
VITE_SENTRY_DSN_FRONTEND=
VITE_LOGROCKET_APP_ID=
VITE_LUXE_URL=http://localhost:3000
VITE_GOOGLE_CLIENT_ID=""
```

Para local con estas dos configuradas basta:
```bash
VITE_GOOGLE_CLIENT_ID="somekey-somekey.apps.googleusercontent.com"
VITE_LUXE_URL=http://localhost:3000
```

Donde:
- `VITE_LUXE_URL` es el servidor de Rails corriendo el repo backend

### Backend

Hay que configurar estas ENVs:

> [!Warning]
> Clave que `ECHO_DOMAIN` no termine en `/`. Sino podría haber problema de CORS.

```bash
REVIEWERS_GOOGLE_OAUTH_ID=somekey-somekey.apps.googleusercontent.com
REVIEWERS_GOOGLE_OAUTH_SECRET=somesecret
ECHO_DOMAIN=http://localhost:5173
```

Donde:
- `REVIEWERS_GOOGLE_OAUTH_ID` es el mismo valor que en `VITE_GOOGLE_CLIENT_ID`
- `ECHO_DOMAIN` apunta al servidor dev del cliente frontend

#### Crear Reviewer en Local

```ruby
Reviewer.create(first_name: "Francisco", last_name: "Quintero", email: "francisco.quintero@ideaware.co")
```