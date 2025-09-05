# Para la versión 1.14

## Configurar secrets con gh CLI

Hay que configurar los secrets que las actions usan en los despliegues al VPS.

Son:

- `TARGET_HOST`
- `TARGET_USER`
- `KNOWN_HOSTS`
- `PRIVATE_KEY`

Además hay que configurar el canal de NTFY para recibir las push de los despliegues. La env es `NTFY_TOPIC`.

Ejemplo configurando el de ntfy:
```bash
gh secret set NTFY_TOPIC --body "$CHANNEL_NAME"
```

Ver docs de [GH CLI](https://cli.github.com/manual/gh_secret_set)