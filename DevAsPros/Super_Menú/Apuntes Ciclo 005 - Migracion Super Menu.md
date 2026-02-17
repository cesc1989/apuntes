# Apuntes Ciclo 005 - Migración a Host Hatch

## Descargar la base de datos con scp

Aprovechando que tengo definidos las configuraciones en `~/.ssh/config`. Así copió del servidor en Linode a local:
```bash
scp dap_node:/home/ubuntu/supermenu/db/supermenu_production.sqlite ~/Downloads/supermenu_production.sqlite
```

Y así cargo al servidor en Host Hatch:
```bash
scp ~/Downloads/supermenu_production.sqlite dap_hatch:/home/ubuntu/supermenu/db/supermenu_production.sqlite
```

## Desactivar servicio en dap_node