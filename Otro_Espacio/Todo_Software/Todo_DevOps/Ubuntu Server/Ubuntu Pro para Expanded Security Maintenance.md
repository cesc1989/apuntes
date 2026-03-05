# Ubuntu Pro en VPS para Expanded Security Maintenance

Relacionado: [[Unattended Upgrades]]

Cuando ingreso al VPS dap_node veo esto:
```
Expanded Security Maintenance for Infrastructure is not enabled.

47 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

109 additional security updates can be applied with ESM Infra.
Learn more about enabling ESM Infra service for Ubuntu 20.04 at
https://ubuntu.com/20-04
```

Este VPS ya tiene varios meses sin reiniciar. Esto porque es una versión de Ubuntu que no tiene más actualizaciones.

```bash
last reboot
reboot   system boot  6.10.2-x86_64-li Thu May 29 06:01   still running
reboot   system boot  6.10.2-x86_64-li Sat May 17 06:01 - 06:00 (11+23:58)
```

La recomendación es migrar a una versión más actualizada de Ubuntu o activar [Ubuntu Pro](https://ubuntu.com/pro/).

> [!Note]
> Este servicio es de pago pero para cuentas personales ofrece soporte hasta para 5 máquinas.