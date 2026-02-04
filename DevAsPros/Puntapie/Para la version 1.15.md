# Para la versión 1.15

## Especificar el puerto del VPS en el github action

Agregar `Port 54321` a la configuración del action de despliegue.

## Actualizar gema redis a la versión 5.4.1 o mayor

Para que tenga soporte con redis-server versión 7.x.x

## Actualizar gema Sidekiq a la versión 7.3.0 o mayor

Para compatibilidad con versiones más recientes de la gema redis

## Aplicar cambios de registrations path

Los últimos cambios hechos en enlacito para completar el registro.

Este pull request: https://github.com/cesc1989/enlacito/pull/14

## Montar landing básica

Tomando los cambios más recientes que haya en enlacito.

Consideraciones de la landing básica:

- Navbar con logotipo en la izquierda y botones en la derecha
- Hero simple
- Tres bloques de características
- Bloque de CTA
- Footer con ToS, Hecho en Barranquilla y el copy + año

## Logs de despliegues van a carpeta `deployments/logs`

En vez de tener `deployments` y `deployment_logs` pasar a una estructura donde los logs quedan en `deployments/logs`. Así es más unificada la experiencia y tiene más sentido.

Ya lo hice en CashFlow, CoshiNotes y Enlacito.

Así quedaron las carpetas luego de este cambio:
```bash
tree -L 2 ./
./
├── cashflow
│   ├── app
│   ├── backups
│   ├── db
│   └── deployments
├── coshinotes
│   ├── app
│   ├── backups
│   ├── db
│   └── deployments
├── enlacito
│   ├── app
│   ├── backups
│   ├── db
│   └── deployments
```