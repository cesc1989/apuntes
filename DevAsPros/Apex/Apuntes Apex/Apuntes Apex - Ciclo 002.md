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

Al poner esa línea en el script `003_after_deploy.sh` da este error en github actions:
```
/usr/local/share/chruby/chruby.sh: line 4: PREFIX: unbound variable
```

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
