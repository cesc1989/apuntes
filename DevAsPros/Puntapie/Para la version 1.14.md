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

Ver docs de [GH CLI](https://cli.github.com/manual/gh_secret_set).

Esto debe ser un script en mis dotfiles o del mismo repo y que se pueda correr desde bash.

## Rake que corre todas las rakes

En vez de listar todas las rakes de manera individual, pongamos una que corra todo.

Tomar de SuperMenu: https://github.com/devaspros/supermenu/commit/6b263004891032507b07c92ea8c26aff568fab63

## Copiar archivo de configuración de nginx

Cualquiera de Cashflow, Coshi Notes o Enlacito.

Tomar de Enlacito: https://github.com/cesc1989/enlacito/blob/main/config/nginx.enlacito.production.conf

## Múltiples actualizaciones a scripts de despliegue

- Prefijo numérico en los archivos principales (los primeros 4)
	- Ver commit: https://github.com/devaspros/supermenu/commit/6ad397523985b5815050e4cf19fa5a9f3759dd3f
	- Tener en cuenta el llamado en el github action: https://github.com/devaspros/supermenu/commit/16e122ac4eda0245d6d90662d55c8f8d006e7689
- Reiniciar solo la rails app con `restart.txt`
	- Y evitar reiniciar nginx
	- Ver commit: https://github.com/devaspros/supermenu/commit/0f5a6fdb871b2dc59396c1bb459784e83a000803#diff-c0a2e3e1f5fa9022138e5d4b47247674a80ad6ca2b4c462ed63660ccd3037fcbR30
- Comando `chown` corre con clave sudo
	- Ver commit: https://github.com/devaspros/supermenu/commit/b718aea1c9574ede83c40b666a362e1eaa0df70d
- Nombre de la carpeta de despliegue en variable
	- Ver commit: https://github.com/cesc1989/cashflow/commit/96c2b7e434753af653237ba8464995acb6d30bee

## Quitar el warning de usar sqlite en production

Tomar de SuperMenu: https://github.com/devaspros/supermenu/commit/aefb7da30e3283f42eafa2da49ddfb6524d2986c

## Notificar despliegues con NTFY

Tomar de Cashflow: https://github.com/cesc1989/cashflow/commit/183fb87e1aa5868dcafeac902c79d61ae03db814

Ver [[adr-001-notificacion-de-estado-despliegues]]

# Configuración de Vite + Despliegue

## Para despliegues

- Cargar `nvm`: https://github.com/devaspros/supermenu/commit/2a05fea754529bc09860d2d11beeccfb32ad7b44
- Incluir instalación de paquetes `npm` usando `npm ci`
	- Ver commit: https://github.com/devaspros/supermenu/commit/0f5a6fdb871b2dc59396c1bb459784e83a000803