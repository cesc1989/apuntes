# Notas del RFC para Estandarizar los Logs en todo Luna

Peter Busche propuso estandarizar los logs de todas las aplicaciones en un formato JSON y en un grupo de niveles. De esa forma tendrá más sentido cada registro de logs y también será sencillo hacer consultas sobre estos.

Sin embargo, la propuesta implica cambiar los logs a JSON. Esto es algo que no he visto en Ruby ni Rails. ¿Hay algo para esto?

## Logger custom a partir de ActiveSupport::Logger::SimpleFormatter

Esto en [Stack Overflow](https://stackoverflow.com/a/29855485/1407371):
```ruby
class MySimpleFormatter < ActiveSupport::Logger::SimpleFormatter
  def call(severity, timestamp, _progname, message)
    { 
      type: severity,
      time: timestamp,
      message: message
    }.to_json
  end
end
```

To configure your new class you'd need to add a config line:
```ruby
config.log_formatter = MySimpleFormatter.new
```

El post es viejo pero la configuración sigue vigente en Rails. [Ver](https://guides.rubyonrails.org/configuring.html#config-log-formatter).

## Gema Lograge

Esta [gema](https://github.com/roidrage/lograge) está en uso en:

- backend: `gem "lograge", "~> 0.13.0"`
- therapist-signup: `gem "lograge", "~> 0.13.0"`
- clinical-dashboard: no está pero pronto este repo se mezclará con backend

Aquí un [gist donde muestran para Airbrake y Lograge](https://gist.github.com/Marinlemaignan/d3811105f098e0fa56af).

```ruby
MAP::Application.configure do
  config.lograge.enabled = true
  config.lograge.keep_original_rails_log = false
  config.lograge.logger = ActiveSupport::Logger.new "#{Rails.root}/log/#{Rails.env}.log" 
  
  config.lograge.formatter = Lograge::Formatters::Json.new

  config.lograge.custom_options = ->(event) {
    {
      time:  %Q('#{event.time}'),
      remote_ip: event.payload[:ip],
      current_user: event.payload[:current_user],
      current_administrator: event.payload[:current_administrator],
      params: event.payload[:params]
    }
  }
end
```

Así se ve un ejemplo:
```json
{"method":"GET","path":"/v1/therapists/078e6da6-24d8-4314-aab0-525fd05f0a0a","format":"json","controller":"Api::V1::TherapistsController","action":"show","status":404,"allocations":2727,"duration":34.01,"view":0.24,"db":39.69,"time":"'155122551.982'","remote_ip":null,"current_user":null,"current_administrator":null,"params":{"format":"json","controller":"api/v1/therapists","action":"show","id":"078e6da6-24d8-4314-aab0-525fd05f0a0a","therapist":{}}}
```