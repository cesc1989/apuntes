# Sidekiq como Demonio

Etiquetas: #sidekiq_cron 

Relacionado [[Apuntes_Ciclo_17_-_Sidekiq_-_Cash_Flow]]

## Cómo ver el estado cuando hay varios servicios Sidekiq en el mismo servidor

Para el caso de Cashflow se ven así:
```bash
systemctl --user status sidekiq.service
```

Para el caso de Enlacito se ven así:
```bash
systemctl --user status enlacito.sidekiq.service
```

