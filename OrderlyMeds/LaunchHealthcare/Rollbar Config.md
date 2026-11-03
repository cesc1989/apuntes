# Rollbar Config

## Crear ROLLBAR_DEPLOY_TOKEN como env en el Repo

Me tocó por terminal porque desde la UI no podía. Lo hice así para cada environment:
```bash
gh secret set ROLLBAR_DEPLOY_TOKEN --env dev -R om-haas-platform/om-haas-api
gh secret set ROLLBAR_DEPLOY_TOKEN --env qa -R om-haas-platform/om-haas-api
gh secret set ROLLBAR_DEPLOY_TOKEN --env stg -R om-haas-platform/om-haas-api
gh secret set ROLLBAR_DEPLOY_TOKEN --env prod -R om-haas-platform/om-haas-api
```

El valor se pasa por la terminal. Se queda esperando hasta que se ingresa.

Para revisar:
```bash
gh secret list --env dev -R om-haas-platform/om-haas-api
NAME                  UPDATED
ROLLBAR_DEPLOY_TOKEN  less than a minute ago
```

## Crear evento en Rollbar para activar enviroment

No podía elegir el env `dev` al configurar los rules para las notificaciones a Slack. Tocó crear un evento manualmente. Así:
```bash
read -s -p "Token dev de Launch API: " RB_TOKEN; echo

curl -s -X POST https://api.rollbar.com/api/1/item/ \
  -H "X-Rollbar-Access-Token: $RB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"data":{"environment":"dev","level":"info","body":{"message":{"body":"HEGR-1087 environment registration"}}}}'

unset RB_TOKEN
```

Donde la primera línea espera a que se ingrese el token en la terminal.