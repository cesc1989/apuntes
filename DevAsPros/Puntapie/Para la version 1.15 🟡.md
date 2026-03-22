# Para la versión 1.15

## Actualizaciones de Versiones 🆙

- Subir Rails a 7.2.3 para quedar cerca al 8. ✅
- Subir a Ruby 3.4.8: ver CashFlow y CoshiNotes. ✅

### Gemas

- Actualizar gema redis a la versión 5.4.1 o mayor ✅
	- Para que tenga soporte con redis-server versión 7.x.x
- Actualizar gema Sidekiq a la versión 7.3.0 o mayor ✅
	- Para compatibilidad con versiones más recientes de la gema redis
- Actualizar sqlite3 a 2.9.0 ✅
	- Ver CashFlow y CoshiNotes.
- Actualizar FactoryBot a 6.5.0 ✅
	- Para prevenir problema con la librería `observer` que fue removida en Ruby 3.4

## Especificar el puerto del VPS en el Github action

Agregar `Port 54321` a la configuración del action de despliegue.

## Aplicar cambios de registrations path 🟡

Los últimos cambios hechos en enlacito para completar el registro.

Este pull request: https://github.com/cesc1989/enlacito/pull/14

Incluye:
- Registro activado
- Parámetros adicionales habilitados
- Controlador de devise generado
- Mínimo de contraseña desde 10

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

## Actualizar README de carpeta de despliegues

Para que se refleje la nueva estructura de carpetas y actualizar el comando de crear las carpetas del proyecto en el VPS.

## Actualiza script de despliegue para que reinicie Sidekiq 

La versión actual solo lo arranca pero no lo actualiza por lo tanto el código nuevo no se refleja en el job.

Ver: https://github.com/cesc1989/enlacito/commit/aac18e7b94922c03be124920c33a5437c9622836

Y: https://github.com/cesc1989/enlacito/commit/5928bb510c9255118e690af8c729570ea1a4ed73

## Indica la versión de ruby esperada con chruby durante despliegue

Ver: https://github.com/cesc1989/cashflow/commit/e031840a2ba76e80d9159633460c91900b41da17

Para evitar problemas durante upgrades de Ruby es mejor siempre especificar. Así se limpia la elección de chruby en el `.profile`.

## Trae clase ExceptionLogger

Para simplificar el envío de errores a Sentry/Bugsink. Commit en Cash Flow https://github.com/cesc1989/cashflow/commit/73861554a604d10fe05108ebf0bda63767cd9453

## Cambiar clave de usuario Admin

Todos los proyectos que he creado a partir de Puntapie tienen la misma clave. Hay que cambiar al generar uno nuevo o hacer que sea auto generada.

## Aplicar cambios de confirmación de correo 📨

Todo lo que se hizo para Enlacito.

- Migración y otros: https://github.com/cesc1989/enlacito/commit/709e5200f0b62a93eceeccf9e20355b331aaa303
- Ajustes en environments: https://github.com/cesc1989/enlacito/commit/8fc7dac6006ad9709d8a850e63a84deab25361e4
- Ajustes para las pruebas:
	- Factories: https://github.com/cesc1989/enlacito/commit/ef1854fac4a94e59a4f301bdd334defe33044a15
	- envs: https://github.com/cesc1989/enlacito/commit/69c5c5d2144ca961b8029ccfbb9796c127d3f7f1
- Configuración de Resend: https://github.com/cesc1989/enlacito/commit/09810c855a1def352b34bba03a986bfdcab7b047
- Ajustes en Devise: https://github.com/cesc1989/enlacito/commit/d906b5ab275a79bab8a5beab238bf3705ab0c3a2
- Más ajustes: https://github.com/cesc1989/enlacito/commit/e367c09bbbcea4be0eece3b6ace4e7f60a5f9555
- Finalmente, correo con buenos estilos y preview para revisar: https://github.com/cesc1989/enlacito/commit/37637121b2193cd1772b7f6d03930c4c73348aa6

## Agregar todas las vistas de Devise para no joder más con esto

Ver: https://github.com/cesc1989/enlacito/commit/a555bc57e92b95350c5c466abcfbb0701fe819ce