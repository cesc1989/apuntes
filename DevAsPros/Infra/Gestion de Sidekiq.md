# Gestión de Sidekiq y Comandos

Etiquetas: #sidekiq_cron 

## Comando para ver el estado cuando hay varios servicios Sidekiq en el mismo servidor

### Cash Flow

```bash
systemctl --user status cashflow.sidekiq.service
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