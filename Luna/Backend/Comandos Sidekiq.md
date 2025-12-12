# Comandos para Revisar Colas en Sidekiq

Etiquetas: #luna_help_desk 

Comandos varios para poder revisar qué pasa con los jobs o colas en Sidekiq desde una consola de Rails.

## Ver encolados de un Job en específico

```ruby
Sidekiq::Queue.new.select { |job| job.klass == "MarketplaceSyncCarePlanWorker" }.each do |job|
  puts "Care Plan ID: #{job.args.first}, Enqueued at: #{Time.at(job.enqueued_at)}"
end; nil
```

Nota:
- Solo cambiar el nombre del worker.
- El `; nil` al final es para que la salida solo sean los puts.

