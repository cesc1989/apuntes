# Apuntes Apex - Ciclo 002

# Mejoras a scripts de despliegue

Quiero fortalecer los scripts de despliegue. He visto muchos casos donde completan con éxito pero en los logs hubo fallos.

## Usando pipefail en scripts de despliegue

DeepSeek recomendó esto:
```bash
#!/bin/bash

set -euo pipefail
```

En cada script de despliegue.

### Error de chruby `PREFIX: unbound variable`

> [!Note]
> Hay problema de chruby con `set -u`
>
> Ver: https://github.com/postmodern/chruby/issues/417

> [!Important]
> Esto podría arreglarse si se deja de cargar el `.profile`
>
> Así lo hace Enlacito.

Al poner esa línea en el script `003_after_deploy.sh` da este error en github actions:
```
/usr/local/share/chruby/chruby.sh: line 4: PREFIX: unbound variable
```

> [!Important]
> La solución fue no usar `-u` porque chruby no juega bien con eso.

Se han probado varias cosas pero nada parece resultar. Probé todo esto.

**Exportar PREFIX**
```bash
export PREFIX="${PREFIX:-/usr/local}"

. /home/ubuntu/.profile
```

**Desactivar -u y reactivarlo**
```bash
set +u
. /home/ubuntu/.profile
set -u
```

Combinar las anteriores
```bash
set +u
export PREFIX="${PREFIX:-/usr/local}"
. /home/ubuntu/.profile
set -u
```

Set después de cargar el `.profile`
```bash
. /home/ubuntu/.profile

set -eo pipefail
```

### Detalle Importante: scripts no llegaban al servidor 🔑

> [!Danger]
> El problema es que todos los cambios que mandé nunca fueron ejecutados porque el servidor se quedó con los archivos donde el despliegue fallaba.
> 
> Cada vez que se corría el action del despliegue, se estaban ejecutando los scripts dañados.

Si agrego este paso al despliegue puedo copiar los scripts antes de ejecutarlos:
```yml
- name: Upload scripts to VPS
	run: |
		scp -r scripts/* supermenu_cloud:~/supermenu/deployments/api-release/scripts/  
```

> [!Warning]
> Esto tiene un problema. Es que modifica los archivos y causa que el pull falle porque "hay cambios" en el repo git que tiene el servidor.