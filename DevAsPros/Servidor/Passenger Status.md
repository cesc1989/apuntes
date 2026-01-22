# Explicando el comando `passenger-status`

Estaba usando este comando para revisar el estado de los diferentes procesos que están en el VPS básico. Tiene bastante información y quiero saber qué significa cada cosa.

Docs: https://www.phusionpassenger.com/docs/advanced_guides/troubleshooting/nginx/overall_status_report.html

## Comando

```bash
sudo passenger-status
```

Salida ejemplo:
```bash
sudo passenger-status
Version : 6.0.23
Date    : 2026-01-21 23:33:32 +0000
Instance: Zzc9mmHw (nginx/1.18.0 Phusion_Passenger/6.0.23)

----------- General information -----------
Max pool size : 6
App groups    : 4
Processes     : 6
Requests in top-level queue : 0

----------- Application groups -----------
/home/ubuntu/cashflow/app (production):
  App root: /home/ubuntu/cashflow/app
  Requests in queue: 0
  * PID: 2877731   Sessions: 0       Processed: 3       Uptime: 7m 44s
    CPU: 0%      Memory  : 79M     Last used: 1m 0s ago
  * PID: 2877750   Sessions: 0       Processed: 0       Uptime: 7m 43s
    CPU: 0%      Memory  : 72M     Last used: 7m 43s ago

/home/ubuntu/coshinotes/app (production):
  App root: /home/ubuntu/coshinotes/app
  Requests in queue: 0
  * PID: 2877533   Sessions: 1       Processed: 5       Uptime: 8m 19s
    CPU: 0%      Memory  : 128M    Last used: 5m 33s ago
  * PID: 2877558   Sessions: 0       Processed: 21      Uptime: 8m 19s
    CPU: 0%      Memory  : 122M    Last used: 1m 56s ago

/home/ubuntu/supermenu/app (production):
  App root: /home/ubuntu/supermenu/app
  Requests in queue: 0
  * PID: 2877811   Sessions: 0       Processed: 1       Uptime: 7m 41s
    CPU: 0%      Memory  : 127M    Last used: 7m 41s ago

/home/ubuntu/enlacito/app (production):
  App root: /home/ubuntu/enlacito/app
  Requests in queue: 0
  * PID: 2877780   Sessions: 0       Processed: 2       Uptime: 7m 42s
    CPU: 0%      Memory  : 114M    Last used: 7m 42s ago
```

## Detalle

Este comando reporta sobre lo siguiente:
- Qué aplicaciones está Passenger sirviendo
- Qué procesos de aplicaciones está Passenger gestionando y su estado con respecto a CPU y uso de memoria
- El estado de las colas de peticiones
- Cualquier petición activa

**Requests in top-level queue**

Los docs dicen que este número debe estar siempre cerca a cero o ser cero. Lo contrario es indicativo de un problema. Probablemente un bug de Passenger.

**/home/ubuntu/cashflow/app (production)**

Este es un application group. Hasta el próximo que se diferencia por empezar por un path.

### Procesos

Cada PID es un proceso que puede aceptar y procesar peticiones. Aquí para CoshiNotes hay dos procesos:
```bash
/home/ubuntu/coshinotes/app (production):
  App root: /home/ubuntu/coshinotes/app
  Requests in queue: 0
  * PID: 2877533   Sessions: 1       Processed: 5       Uptime: 8m 19s
    CPU: 0%      Memory  : 128M    Last used: 5m 33s ago
  * PID: 2877558   Sessions: 0       Processed: 21      Uptime: 8m 19s
    CPU: 0%      Memory  : 122M    Last used: 1m 56s ago
```

- `PID`: ID de proceso del sistema operativo.
- `Sessions`: número de peticiones que el proceso está gestionando.
- `Processed`: número de peticiones que el proceso ha gestionado.
- `Last used`: el tiempo cuando una petición fue enviada a este proceso por última vez.