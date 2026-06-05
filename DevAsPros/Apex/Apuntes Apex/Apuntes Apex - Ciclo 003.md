# Apuntes Apex - Ciclo 003

## Error de pull en script de despliegue

Salía este error en el paso del pull desde el script de despliegue:
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

El error se daba porque el remoto trataba de acceder a github.com de la forma tradicional. Sin embargo, para los scripts del VPS se debe acceder usando el alias configurado en `~/.ssh/config`.

```bash
cat ~/.ssh/config
Host gh
  HostName github.com
  User git
  IdentityFile ~/.ssh/gh_ubuntu
```

Así que al clonar debe hacerse así:
```bash
git clone git@gh:devaspros/postlane-backend.git .
```

Para corregir hice este comando:
```bash
git remote set-url origin git@gh:devaspros/postlane-backend.git
```

Luego pude verificar con:
```bash
git remote -v
origin	git@gh:devaspros/postlane-backend.git (fetch)
origin	git@gh:devaspros/postlane-backend.git (push)
```

## Errores de bundle con flags

Obtuve estos errores:
```
The `--deployment` flag has been removed because it relied on being remembered across bundler invocations, which bundler no longer does. Instead please use `bundle config set deployment true`, and stop using this flag

The `--path` flag has been removed because it relied on being remembered across
bundler invocations, which bundler no longer does. Instead please use `bundle
config set path '/home/ubuntu/postlane/deployments/api-gems/bundle'`, and stop
using this flag
```

Así que usé los comandos que ahí dice para configurar el archivo en `.bundle/config` que tiene el repo en el VPS.

> [!Important]
> A la fecha, ese archivo en local no lo tengo configurado porque esos valores no sirven para development.

Así quedó:
```bash
---
BUNDLE_DEPLOYMENT: "true"
BUNDLE_PATH: "/home/ubuntu/postlane/deployments/api-gems/bundle"
BUNDLE_WITHOUT: "development:test"
```

