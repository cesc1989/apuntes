# Gestión de Sidekiq, Jobs y comandos systemctl

Etiquetas: #sidekiq_cron 

General. Recargar demonio de systemd:
```bash
systemctl --user daemon-reload
```

## Comando para ver el estado cuando hay varios servicios Sidekiq en el mismo servidor

### Cash Flow

#### Sidekiq

```bash
systemctl --user status cashflow.sidekiq.service
```

Otros:
```bash
systemctl --user stop cashflow.sidekiq.service
systemctl --user disable --now cashflow.sidekiq.service
```

#### Jobs

Estado:
```bash
systemctl --user status cashflow.jobs.service
```

Reiniciar servicio jobs:
```bash
systemctl --user restart cashflow.jobs.service
```

### Enlacito

```bash
systemctl --user status enlacito.sidekiq.service
```

## Comando para reiniciar Sidekiq de un servicio especifico

### Enlacito

```bash
systemctl --user restart enlacito.sidekiq.service
```

# Comandos de verificación de servicio deshabilitado

Para saber si el servicio fue deshabilitado.

```bash
systemctl --user status cashflow.sidekiq.service
```
> Debería responder algo como Unit cashflow.sidekiq.service could not be found. — eso confirma que se borró bien.

