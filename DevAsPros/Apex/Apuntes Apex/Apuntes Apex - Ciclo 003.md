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