# Apuntes Ciclo 23 - Logrotate

# Configuración de logrotate en Rails apps

En servidores VPS, se recomienda hacer rotación de los logs para evitar que crezcan de manera desmedida y afectar al servidor. Cuando un archivo de log crece demasiado, puede ocupar todo el espacio de disco y no dejar que ningún otro proceso se ejecute.

Para Rails hay dos formas de hacer rotación de logs. Una es con un archivo que use la sintaxis de logrotate y la otra es mediante la librería Logger de Ruby.

## Rotación con logrotate

El archivo debe seguir estas reglas:
```
/home/ubuntu/cashflow/app/log/*.log {
  weekly
  missingok
  rotate 7
  compress
  delaycompress
  notifempty
  copytruncate
}
```

A continuación explico cada línea.

```
/home/ubuntu/cashflow/app/log/*.log
```

Esta es la ruta donde está el/los archivos de log. Dentro de las llaves se configura la rotación:

- `weekly`: hace la rotación cada semana.
- `missingok`: no explota si no hay archivo de log
- `rotate 7`: mantiene 7 archivos de rotación
- `compress`: crea el archivo con compresión gzip
- `delaycompress`: retrasa la compresión hasta la segunda rotación. Es decir que habrá un archivo de log, uno rotado sin comprimir, el resto de rotación compreso.
- `notifempty`: no hace la rotación si el archivo está vacío
- `copytruncate`: esta es importante para Rails. Con esta forma primero se crea una copia del log y después se vacía. Así no hay que reiniciar el servidor de Rails.

## Rotación con clase `Logger` de Ruby

Esto me sorprendió mucho. La clase Logger de Ruby tiene una opción para indicar cuándo hacer la rotación.

Así quedó la configuración en `config/environments/production.rb`:
```ruby
config.logger = ActiveSupport::Logger.new("log/production.log", "daily")
```

Con eso, a media noche, hará la rotación del log:
```
-rw-r--r--  1 ubuntu ubuntu 1M Mar 13 14:04 production.log
-rw-rw-r--  1 ubuntu ubuntu 1M Mar  1 00:05 production.log.20250312
```

## Enlaces

- [Managing Logs with Logrotate](https://serversforhackers.com/c/managing-logs-with-logrotate)
- [Ruby on Rails production log rotation](https://stackoverflow.com/questions/4883891/ruby-on-rails-production-log-rotation)
- [Rotating Rails Production Logs with LogRotate](https://gorails.com/guides/rotating-rails-production-logs-with-logrotate)

# Pruebas de mailers cuando hay uno o varios formatos

Antes, enviaba el correo de los Chores en versión HTML y Texto plano. Tenía dos archivos en las vistas:

- `app/views/chores_mailer/upcoming_chores.text.erb`
- `app/views/chores_mailer/upcoming_chores.html.erb`

En ese caso podía probar el cuerpo del correo de esta forma para la versión HTML:
```ruby
expect(mail.html_part.body).to include(chore.name)
```

Sin embargo, una vez eliminé el formato texto plano del mailer eso no es posible. [Así lo explican las guías](https://guides.rubyonrails.org/v7.0/testing.html#the-basic-test-case):

> `email.body.to_s` is present when there's only one (HTML or text) part present. If the mailer provides both, you can test your fixture against specific parts with `email.text_part.body.to_s` or `email.html_part.body.to_s`.

En su lugar tuve que cambiar la prueba:
```diff
- expect(mail.html_part.body).to include(chore.name)
+ expect(mail.body).to include(chore.name)
```

No pude encontrar explicación de esto en la gema Mail.