# Gestión de Jobs con comandos systemctl

Etiquetas: #sidekiq_cron 

General. Recargar demonio de systemd:
```bash
systemctl --user daemon-reload
```

## Cash Flow

### Sidekiq

```bash
systemctl --user status cashflow.sidekiq.service
```

Otros:
```bash
systemctl --user stop cashflow.sidekiq.service
systemctl --user disable --now cashflow.sidekiq.service
```

### Jobs

Estado:
```bash
systemctl --user status cashflow.jobs.service
```

Reiniciar servicio jobs:
```bash
systemctl --user restart cashflow.jobs.service
```

## Enlacito

### Sidekiq

```bash
systemctl --user status enlacito.sidekiq.service
```

Reiniciar:
```bash
systemctl --user restart enlacito.sidekiq.service
```

Otros:
```bash
systemctl --user stop enlacito.sidekiq.service
systemctl --user disable --now enlacito.sidekiq.service
```

### Jobs



# Comandos de verificación de servicio deshabilitado

Para saber si el servicio fue deshabilitado.

```bash
systemctl --user status cashflow.sidekiq.service
```
> Debería responder algo como "Unit cashflow.sidekiq.service could not be found". Eso confirma que se borró bien.

