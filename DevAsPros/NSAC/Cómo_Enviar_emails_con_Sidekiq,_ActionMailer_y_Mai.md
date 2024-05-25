# Cómo Enviar emails con Sidekiq, ActionMailer y Mailgun
En [este gist](https://gist.github.com/cesc1989/cc6ab1150ba698019d0710e94ea0d851) expongo bastante detalle sobre la configuración y algunas notas al respecto. Acá exploro un poco más.

# Sobre ActionMailer y Sidekiq adapter

[Documentación](https://github.com/mperham/sidekiq/wiki/Active-Job#action-mailer).

ActiveJob le brinda a los mailers el método `#deliver_later`. Si queremos usar este método con Sidekiq, hay que configurar la gema como el adaptador.

    # config/application.rb
    class Application < Rails::Application
      # ...
    
      config.active_job.queue_adapter = :sidekiq
    end

Esto también nos trae una diferencia crucial y es el uso de Global ID.

Global ID es lo que permite que podamos pasar instancias de objetos a los mailers al usar `#deliver_later`.

    UserMailer.with(user: @user).welcome_email.deliver_later

La documentación también específica que hay que indicar la cola `mailers` al arrancar Sidekiq:

    bundle exec sidekiq -q default -q mailers
# Sobre ActionMailer y Sidekiq delayed extensions

[Documentación](https://github.com/mperham/sidekiq/wiki/Delayed-extensions#actionmailer).

Se pueden usar los métodos de Sidekiq:

- `delay`
- `delay_for(interval`)
- y `delay_until(time)`

en las clases de ActionMailer y ActiveRecord siempre y cuando se habiliten. Para hacerlo, crea un inicializador:

    # config/initializers/sidekiq.rb
    Sidekiq::Extensions.enable_delay!

Con esto activado, puedes ejecutar el mailer así:

    # app/controllers/pqr_controller.rb
    class Public::PqrsController < ApplicationController
      def create
        PqrMailer.delay.pqr_link(@pqr.id)
      end
    end

Y en el mailer se instancia de nuevo el registro.

Esto es así porque Sidekiq prefiere que se envíen primitivas a sus workers y no instancias de clases.

> **It is recommended to avoid passing an object instance to mailer methods.** Instead, pass an object id and then re-instantiate the object in the mailer method


# Sobre worker en Heroku

Resulta que tenía el Procfile configurado para correr Heroku:

    release: bundle exec rails db:migrate
    web: rails server
    worker: bundle exec sidekiq

pero no se estaba ejecutando nada cuando revisaba la web de Sidekiq. Todo estaba en cola.

Hasta que me di cuenta gracias a [esta respuesta](https://stackoverflow.com/a/56382769/1407371) en Stack Overflow, punto 6, de que tenía el worker apagado en Heroku. Presioné el lápiz, prendí el worker y Sidekiq empezó a trabajar.

![hay que prender el worker](https://paper-attachments.dropbox.com/s_F75F017BA45976A4CACE7D74639CE72B01AD52E8330EDC9C2DFD79A4CA5F981F_1620747564787_image.png)



# Sobre el error de EOFError

Al intentar mandar correos sale este error:

    ActionMailer::MailDeliveryJob jid=63095e2383ecd1023f8041ff elapsed=5.022 INFO: fail
    
    EOFError: end of file reached
    /app/vendor/ruby-2.7.1/lib/ruby/2.7.0/net/protocol.rb:225:in `rbuf_fill'

NPI. La configuración es muy cercana a la de [este artículo](https://www.leemunroe.com/send-automated-email-ruby-rails-mailgun/). En [este otro](https://hixonrails.com/ruby-on-rails-tutorials/ruby-on-rails-action-mailer-configuration/#ruby-on-rails-action-mailer-mailgun-configuration) también veo una configuración similar, sin embargo, decidí cambiar el puerto al 587.

¡Y eso fue! Tuve que cambiar el puerto o que fuera leído directo desde la configuración… O tal vez porque era un string y no un integer?

